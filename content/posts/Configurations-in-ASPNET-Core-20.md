---
title: "Configurations in ASP.NET Core 2.0"
date: 2017-10-03T13:55:57+04:30
draft: false
---

With new version of ASP.NET Core configuration system has changed a lot. First it is now inject able into the framework. then you can add custom options to the base Configuration class and access it anywhere needed.

In this post I'm going to list all available configurations in ASP.NET Core 2.0 . and to do that I started by injecting configuration using a constructor.

```csharp
public class Startup {
public IConfiguration Configuration { get; }
    public Startup(IConfiguration config)
        {
            Configuration=config;
        }
}
```

From now on we can use **Configuration** anywhere in the **Startup** class. My mission is to print all keys and values of Configuration object. I use **Configure** method in startup and create an In-line middleware.

```csharp
public void Configure (IApplicationBuilder app) {
            app.Run (async (context) => {
                var keyList = Configuration.GetChildren();
                var body = "";
                foreach (var item in keyList)
                {
                   body += Environment.NewLine +
                   item.Key + " : " + item.Value;
                }
              await context.Response.WriteAsync(body);
            });
        }
```

First I used **GetChildren()** method of Configuration object to find all the key/value pairs of configuration system and created a body of strings. then passed the body to the middleware. The result is a long list of all settings you have on a Windows OS.

## Conclusion

It's really fascinating to work with asp.net core configuration. there are so many features. my favorite is that you can create Class and add it to configurations system and with no extra code it is available in **appsettings.json** file.
