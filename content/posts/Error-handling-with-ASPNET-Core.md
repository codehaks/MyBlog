---
title: "Error Handling With ASP.NET Core"
date: 2017-09-15T13:52:15+04:30
draft: false
---

Error handling is real and it's serious. In ASP.NET Core it's much more easier to handle errors. in this post I'm digging into error handling and trying to get some errors !

## Middlewares

Like many other features in ASP.NET Core, Error handling is done through Middlewares, there is **UseDeveloperExceptionPage()** and **UseExceptionHandler()** to start with. The first one is used to show detailed error information for developers and the second is to redirect users to error page when something goes wrong. Usually you would not use them together, because they override each other. it's a good practice to use a control flow based on Environment.

```csharp
if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/error");
    }
```

## Sample project

First let's create a sample project that simulates an Exception. Here is my startup.cs using the default web template.

```csharp
public class Startup {
        public void Configure (IApplicationBuilder app) {
            app.UseDeveloperExceptionPage();
            app.UseExceptionHandler("/error");
            app.Use (async (context, next) => {
                if (context.Request.Path == "/throw") {
                    throw new Exception ("Something went wrong!");
                }
                await context.Response.WriteAsync
                (Environment.NewLine + " No Error");
                await next.Invoke();
            });
            app.Run (async (context) => {
                await context.Response
                .WriteAsync(Environment.NewLine + context.Request.Path);
            });
        }
    }
 ```

As you see I used **UseDeveloperExceptionPage** and **UseExceptionHandler** in the same code flow, so user only gets redirected to error page and no error details would be visible to anyone.

## Error details

To show detailed error page we can remove or comment out the **UseExceptionHandler** middleware.

```csharp
//app.UseExceptionHandler("/error");
```

## Throwing exception

To test the exception handler I created an In-line middleware with **app.Use()** method.Now When users sends request to "/throw" url gets an error with detailed information.

To redirect user to error page we need to replace **UseDeveloperExceptionPage** with **UseExceptionHandler** . then we can see **/error** gets written into the response.

## Conclusion

There is much into error handling in ASP.NET Core. The real use of error handling is in MVC projects where there is a lot more code.
