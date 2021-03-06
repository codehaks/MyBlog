---
title: "A Vuejs CRUD Project With ASP NET Core"
date: 2017-09-12T13:42:34+04:30
draft: false
---

I enjoy working with Vue.js. It's just a good SPA framework, it has everything I like about a javaScript framework. Today I created a sample project with Vue.js and ASP.NET Core MVC 2.0 that has all CRUD operations. I want to share the code here.

## Project BugVue

This is a simple app for tracking bugs. You can Create, Read, Update and Delete the in UI and it all happens in the same view thanks to Vue.js and jQuery AJAX function.

### Getting started

The project has two main parts. a **Back-end** which I used ASP.NET Core MVC 2.0 and **Front-end** which I used Vue.js 2.x . I started with the empty template and added features as I needed. For database I used SQLite and EntityFramework core.

Bug.cs goes like this :

```csharp
public class Bug
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
    }
```

### Database

Then we need a DbContext object to create the database using EntityFramework. 

```csharp
public class BugDb : DbContext
    {
        public BugDb(DbContextOptions options) : base(options)
        {
        }
        public DbSet<Bug> Bugs { get; set; }
    }
```

There is a DbContextOptions injected into to constructor which is used for sending options to DbContext object without changing the class itself. later we add a DbContext as a service in startup.cs file. 

```csharp
public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<BugDb>(options => options.UseSqlite("DataSource=bug.db"));
            services.AddMvc();
        }
        public void Configure(IApplicationBuilder app)
        {
            app.UseStaticFiles();
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
```

I use **options** to pass Sqlite and its connection string here. Then I added MVC service and used the middleware to setup the default routing.

### Web API

Now I have to add a controller to manage ajax calls. This is just a simple web api.

```csharp
public class BugController:Controller
    {
        private readonly BugDb _db;
        public BugController(BugDb db)
        {
            _db = db;
        }
        [HttpGet]
        [Route("/bug")]
        public IActionResult Get()
        {
            var model = _db.Bugs;
            return Ok(model);
        }
        [Route("/bug")]
        [HttpPost]
        public IActionResult Post(Bug model)
        {
            _db.Bugs.Add(model);
            _db.SaveChanges();
            return Ok(model);
        }
        [Route("/bug")]
        [HttpPut]
        public IActionResult Put(Bug model)
        {
            var bug = _db.Bugs.Find(model.Id);
            bug.Name = model.Name;
            bug.Description = model.Description;
            _db.SaveChanges();
            return Ok(model);
        }
        [Route("/bug")]
        [HttpDelete]
        public IActionResult Delete(Bug model)
        {
            var bug = _db.Bugs.Find(model.Id);
            _db.Bugs.Remove(bug);
            _db.SaveChanges();
            return Ok(model);
        }
    }
```

As you can see there is GET,POST,PUT,DELETE methods withe the same routing. There is no validation, I skipped that part due to simplicity.

## User Interface

The UI is just a simple Table with some buttons. I also used bootstrap modals for creating and editing forms.

```html
<div id="app">
    <button class="btn btn-primary" v-on:click="showNewBugModal"> Add new bug ... </button>
    <table class="table">
        <thead>
            <tr>
                <th>#</th>
                <th>Name</th>
                <th>Description</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(bug,index) in bugs">
                <td>{{index+1}}</td>
                <td>{{bug.name}}</td>
                <td>{{bug.description}}</td>
                <td>
                    <button class="btn btn-danger" v-on:click="removeBug(bug,index)">Remove</button>
                    <button class="btn btn-default" v-on:click="showEditModal(bug,index)">Edit</button>
                </td>
            </tr>
        </tbody>
    </table>
```

### Client-Side with Vue.js

Last but not least, there is a vue object which is used as a ViewModel to manage the UI and events. The two-way binding works simple and easy.

```javascript
<script>
        var app = new Vue({
            el: "#app",
            data: {
                bugs: [],
                name: "",
                description: "",
                bugId: null,
                bugIndex: null
            },
            created: function () {
                this.getBugs();
            },
            methods: {
                showEditModal: function (bug, index) {
                    this.bugId = bug.id;
                    this.bugIndex = index;
                    this.name = bug.name;
                    this.description = bug.description;
                    $("#editBugModal").modal("show");
                },
                editBug: function () {
                    var vm = this;
                    var newBug = {
                        id: vm.bugId,
                        name: vm.name,
                        description:vm.description
                    }
                    $.ajax({ url: "/bug", data: newBug, method: "PUT" })
                        .done(function () {
                            vm.bugs[vm.bugIndex].name = vm.name;
                            vm.bugs[vm.bugIndex].description = vm.description;
                            toastr.success("Success");
                        }).fail(function () {
                            toastr.error("Error");
                        }).always(function () {
                            vm.name = "";
                            vm.description = "";
                            $("#editBugModal").modal("hide");
                        });
                },
                removeBug: function (bug,index) {
                    var vm = this;
                    $.ajax({ url: "/bug", data: bug, method: "DELETE" })
                        .done(function (data) {
                            vm.bugs.splice(index, 1);
                            toastr.success("Success");
                        }).fail(function () {
                            toastr.error("Error");
                        });
                },
                showNewBugModal: function () {
                    $("#addNewBugModal").modal("show");
                },
                addBugs: function () {
                    var vm = this;
                    var newBug = {
                        name: vm.name,
                        description: vm.description
                    }
                    $.ajax({ url: "/bug", data: newBug, method: "POST" })
                        .done(function (data) {
                            vm.bugs.splice(0, 0, newBug);
                            toastr.success("Success");
                        }).fail(function () {
                            toastr.error("Error");
                        }).always(function () {
                            vm.name = "";
                            vm.description = "";
                            $("#addNewBugModal").modal("hide");
                        });
                },
                getBugs: function () {
                    var vm = this;
                    $.ajax({ url: "/bug", method: "GET" })
                        .done(function (data) {
                            vm.bugs = data;
                            //toastr.success("Success");
                        }).fail(function () {
                            toastr.error("Error");
                        });
                }
            }
        });
    </script>
```

## Conclusion

Vue.js is an easy framework to learn. It is light weight and simple to use. No need for NPM or WebPack . you just add the min.js file and go with it. Most of the time that is all you need.

You can download <https://github.com/codehaks/BueVue> here. 
