---
title: "Injecting LiteDb as a service in ASP.NET Core"
date: 2018-10-01T13:35:59+04:30
draft: false
---
# Intoduction
LiteDB is a simple, fast and lightweight embedded .NET document database. LiteDB was inspired by the MongoDB database and its API is very similar to MongoDB's official .NET API.

# Situation
I have been using LiteDb for some of my smaller projects and I've got to tell you that so far it has beed satisfying. But my problem was injecting LiteDb as a service inside ASP.NET Core applications. Usually in ASP.NET Core all buit in services are added like this : 

```csharp
 public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }
```

Of course it's possible to inject your services using "Action" syntax like this :

```csharp
 public void ConfigureServices(IServiceCollection services)
        {
            ServicesConfig.Register(services);
        }

         public static void Register(IServiceCollection services)
        {
           
            services.AddTransient<IProductService, ProductService>();
            services.AddTransient<IOrderService, OrderService>();
        }
```

But thats not what I want. I want to have a neat looking startup. And I have seen a couple of popular frameworks like AutoMapper already using this :

```csharp
     services.AddAutoMapper();
```

## Implemention

I have always wanted to have something similar with my other services.So I tried to implement it for LiteDb.

### First Step

first I used a different class to to keep all necessary configurations separately. For LiteDb we only need a path to create database file.
<script src="https://gist.github.com/codehaks/9ca799a0734f8cc775f0d585febdcca3.js"></script>

### Second Step
Next I added an other class for the actual LiteDb service.

<script src="https://gist.github.com/codehaks/12b528749e07d53a7451f14d9162dada.js"></script>

### Finally

The last step would be to add an extension to *IServiceCollection* interface.


<script src="https://gist.github.com/codehaks/7e19f5e4a545c4f987d244640868c334.js"></script>


Then all we need to do is add "usings" and call *AddLiteDb" in ConfigureServices method at startup.

```csharp
public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
            services.AddLiteDb(@"bug.db");
        }
```

## Conclusion

Extensions in C# usally help with technical needs but in many cases we use them to add cleaner look in code. The best was to create an extension is to move backwards and start from the end.

You can download the source code here : <https://github.com/codehaks/BugPages>
