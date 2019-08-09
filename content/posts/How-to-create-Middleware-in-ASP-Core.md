---
title: "How to Create Middleware in ASP Core"
date: 2017-09-04T13:25:00+04:30
draft: false
---


One of the best features of ASP.NET Core is it's flexible pipeline. You can control what goes into your application's pipeline and add more to it using a *Middleware*.

## SendFile Middleware

This middleware is just a sample project to see how middlewares are created in ASP.NET Core. I tried to keep it is simple as possible,after all this is my first middleware.

## What it does?

You set a file path at startup and SendFile Middleware sends the file back to user in browser. basically a FileResult directly as a response.

```csharp
public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            app.UseSendFile(@"C:\myfile.jpg");
        }
    }
```

It can be any kind of file, and user receives it in browser.

## SendFile middleware explained

The basic parts of a middleware in ASP.NET Core is simple. You have to create a class like this : 

```csharp
internal class SendFileMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly SendFileMiddlewareOptions _options;
        public SendFileMiddleware(RequestDelegate next, IOptions<SendFileMiddlewareOptions> options)
        {
            _options = options.Value;
            _next = next;
        }
        public Task Invoke(HttpContext context)
        {
            if (!string.IsNullOrEmpty(_options.FilePath))
            {
                var file = new Microsoft.Extensions.FileProviders
                .Physical.PhysicalFileInfo(
                    new System.IO.FileInfo(_options.FilePath));
                context.Response.SendFileAsync(file);
                return Task.CompletedTask;
            }
            else
            {
                return Task.CompletedTask;
            }
        }
    }
```

As you can see there is no inheritance involved. You just have to provide a structure. the **_next** property is a request delegate that you use to pass the response to next middleware. There is an **options** parameter. if you want to pass data to your middleware you need an **IOptions** interface. Next there is the **Invoke** method. it takes a HttpContext and creates the response then you pass the response to next middleware. you dont have to pass if your middleware is a short-circuit one.

```csharp
namespace Codehaks.Middlewares.SendFile
{
    public class SendFileMiddlewareOptions
    {
        public string FilePath { get; set; }
    }
}

```

## The extension method

The best way to present your middleware is to create an Extension method and give other developer the option to call it directly with IApplication parameter.

```csharp

public static class SendFileMiddlewareExtension
    {
        public static IApplicationBuilder UseSendFile(
            this IApplicationBuilder builder, 
            SendFileMiddlewareOptions options)
        {
            return
            builder.UseMiddleware<SendFileMiddleware>
            (Options.Create(options));
        }
        public static IApplicationBuilder UseSendFile(
            this IApplicationBuilder builder, 
            string filePath)
        {
            var options = new SendFileMiddlewareOptions()
            {
                FilePath = filePath
            };
            return
            builder.UseMiddleware<SendFileMiddleware>
            (Options.Create(options));
        }
    }
```

## Conclusion

Middlewares can solve many problems,Creating them is fairly easy and working with them is fun. But there are many things to watch for. Like when you change response it the pipeline the next middleware is not allowed to chat it again,there are some reasons for that. but thats the topic for an other post.
SendFile middleware is available as a nuget :

[source,C#]
PM > Install-Package Codehaks.Middlewares.SendFile

you can [get the source code here](https://github.com/codehaks/Codehaks.Middlewares.SendFile)