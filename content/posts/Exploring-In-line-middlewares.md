---
title: "Exploring in Line Middlewares"
date: 2017-09-07T13:35:59+04:30
draft: false
---

In-line middlewares are a simple way to add functionality to ASP.NET Core pipeline without going through a ceremony. In this post I'm playing with app.Run and app.Use and see how these two work.

## Simplest middleware possible

The best way to understand how a middleware works is to remove the noise around it. so, this is the simplest middleware you can build in ASP.NET Core 2.0.

```csharp
app.Run(async (context) =>
{
    await context.Response.WriteAsync("Hello World! 0");
});
```

By the way, that is exactly what you get when you create a new project in Visual studio,using the default template. 

As this was an exploration, I went a step further and found the actual method in source code.

```csharp
public static void
Run(this IApplicationBuilder app, RequestDelegate handler)
        {
            if (app == null)
            {
                throw new ArgumentNullException(nameof(app));
            }
            if (handler == null)
            {
                throw new ArgumentNullException(nameof(handler));
            }
            app.Use(_ => handler);
        }
```

So, basically what **app.Run** does is calling the **app.Use** method. but in this case, **app.Run** does not introduce any **next** parameter, so no middleware after **app.Run** would run. This is a short-circuit middleware.

Now, lets take a look at app.Use method under the hood :

```csharp
public static IApplicationBuilder Use(this IApplicationBuilder app, Func<HttpContext, Func<Task>, Task> middleware)
        {
            return app.Use(next =>
            {
                return context =>
                {
                    Func<Task> simpleNext = () => next(context);
                    return middleware(context, simpleNext);
                };
            });
        }
```

This is also an Extension method,and returning a **RequestDelegate** . so, what is a RequestDelegate !?

```csharp
public delegate Task RequestDelegate(HttpContext context);
```

A request delegate is a function that can process an HTTP request. This function returns a Task indicating completion of the request process.

## Conclusion

We can use **app.Use** to simulate **app.Run** method. Basically the **app.Run** is a simpler version of app.Use. Usually you use **app.Run** the the end of your pipeline because it is a short-circuit middleware and does not **Invoke** next middlewares.

There are at least 4 different ways to show "Hello,world!" using in-line middlewares in ASP.NET Core.

You can see it
<https://gist.github.com/codehaks/61d026c46c0293435cab375290c24980> here.