---
title: "Bare Minimum Hello World"
date: 2017-08-31T13:02:05+04:30
draft: false
---

In this post I'm exploring what is the bare minimum necessary code to show "Hello World!" in an ASP.NET Core 2.0 Application. bear with me!

## The template

To get the bare minimum in the first place you need to create a  project using "ASP.NET Core Empty" template.

## Program.cs

The most important part that you have to change here is the *CreateDefaultBuilder* Method.There are so many parts that you don't need, Like settings a Logging. To do that I used at the source code (Thanks to open-source) and create the host myself.

```csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
namespace BareMinCore
{
    public class Program
    {
        public static void Main(string[] args)
        {
             var builder = new WebHostBuilder()
                .UseKestrel()
                .UseStartup<Startup>()
                .Build();
            builder.Run();
        }
    }
}
```

## Startup.cs

This part is a little tricky. I found out that there is no need for constructor or *ConfigureServices*. But we need the *Configure* method always. Thats how ASP.NET Core 2.0 works anyway. so this is what I end up with :

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
namespace BareMinCore
{
    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            app.Run(async (context) =>
            {
                await context.Response.WriteAsync("Hello World!");
            });
        }
    }
}
```

## Conclusion

It's always good to find out what is there and what we actually need to run the system. It helps us to understand how the system is working and how we can improve it using less resources. You can download the result project at my github page.
