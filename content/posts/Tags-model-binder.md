---
title: "Tags Custom ModelBinder"
date: 2019-08-09T10:49:41+03:30
draft: false
---

In this post we are going to create a custom model binder in ASP.NET Core MVC that converts CSV text into IEnumerable of strings automatically. 

## Model

In MVC a model is where data is being kept while going from controller to view and comming back. In ASP.NET MVC since we are using C# a Model is actually a class with some properties.

## Model Binder

A model binder's job is to map(bind) content in HTTP request to the models. in HTTP data goes back and forth in text format. so we need a way to convert these text data to the corresponding properties. ASP.NET Core has about 17 default model binders.these are used for everyday requirements like converting enums,numbers and arrays to class properties and collections.

## Model Binder Provider

There is a IModelBinderProvider interface which you have to implement if you want to create your own Model Binder.

## Custom Tags Model Binder

For one of my projects I needed to have tags for posts. I used a "Comma-seperated-value" system to differentiate between each word.
so I asked user for an Input like this : "Programming,Web,Patterns". the problem is by default ASP.NET Core Binds the whole text to a property in my class. But I want to have a custom behavior which turns this text in to an Array of strings splitted by ","

## Getting started

To create a custom Tag Helper you need to create a new class and inherit from **IModelBinder** interface:

```csharp
 public class TagsModelBinder : IModelBinder
    {
        public Task BindModelAsync(ModelBindingContext bindingContext)
        {
            // your code goes here
             return Task.CompletedTask;
        }
    }
```

Withoud doing any actual work, You can now start testing your first custom build model binder. Go to the action inside the controller:

```csharp
public IActionResult Get([ModelBinder(typeof(TagsModelBinder))] string tags ,Product model)
        {
            // tags is null here!
            return Ok();
        }
```

if you run the code in debug mode you'll see that putting a break point inside TagsModelBinder will hit every time you post data to the controller. but inside the action the value of **tags** is null. the problem is when you replace the default model binders with your own you have to provide the correct code to to the bindings. ASP NET Core no longer falls back to it's default mechanism in case anything goes wrong.

## Implemention

Here is the full implemention of my custom model binder for Tags :

<script src="https://gist.github.com/codehaks/137d74cd91604bf7985e45be6e851e86.js"></script>

I grab the value inside Tags input field and use **split** method to convert it to **IEnumerable<string>**. Now if you run the app again and send data from form to action, everything works and text data gets converted to an Enumerable of strings.

## Provider

so far we have done a good job of having a custom model binder, But everytime we want to use this model binder we have to use it like an attribute. like **[ModelBinder(typeof(TagsModelBinder))] string tags** which is not so clean. We can solve this issue by adding a Model Binder Provider class and register it to ASP.NET MVC Model binding service,so when ever we have an IEnumerable<string> as an argument type inside actions automatically use it.

<script src="https://gist.github.com/codehaks/0c2c7ebe3009b6a7012ad85a1dbda27b.js"></script>

Then we need to register it in startup.cs file inside ConfigureServices method. 

```csharp
public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc(options =>
            {
                // add custom binder to beginning of collection
                options.ModelBinderProviders.Insert(0, new TagsModelBinderProvider());
            });
        }
```

we use zero index for insert because the order that a provider gets called is important. the first provider that return a value other than null gets used and all others are skipped.

## Finally

Now that we have a provider inplace we can change the argument type from string to IEnumerable<string> :

```csharp
public IActionResult Get(IEnumerable<string> tags ,Product model)
        {
            foreach (var item in tags)
            {
                // tags are converted to collection.
            }
            return Ok();
        }
```

## Conclusion

Usually it's not necessary to create your own custom model binder but sometimes it solves a big problem and helps with a cleaner code and long time maintenance.

You can download the source code here : <https://github.com/codehaks/BindingDemo>
