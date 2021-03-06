---
  title: IStartupFilter
  header:
    subheading: Simplifying component setup with IStartupFilter
  categories:
    - dotnetcore
  tags:
    - csharp
    - dotnetcore
    - asp
---

When registering services in ASP.Net Core, often you have to register 
the services in `Startup.ConfigureServices` and then tell the 
`IApplicationBuilder` to actually use them in `Startup.Configure`.

Take the example of JwtBearer Authentication. You have to remember to call 

```c#
services.AddAuthentication(options => ...)
```

in `ConfigureServices` _and_

```c#
app.UseAuthentication()
```

in `Configure`.

Not only is this open to making mistakes, it scatters the configuration of 
a service across multiple methods. This is not particularly convenient if
you want to distribute an API because consumers of your API don't have one 
place to register and configure your service.

Enter the `IStartupFilter`, part of the `Microsoft.AspNetCore.Hosting.Abstractions` 
package. It defines one method:

```c#
Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next);
```

_Note that this does not apply to the Generic Host, which has a different startup model._

When you call `Build` on your `HostBuilder` (either Generic or Web) the runtime retrieves an `IEnumerable` of `IStartupFilters` from its
`ServiceCollection` and then iterates through them in order invoking the `Configure` method on your app's `ApplicationBuilder`.

As a particularly contrived version, consider a middleware that has its own dependencies.

```c#
public class DemoMiddleware
{

    readonly RequestDelegate _next;
    readonly IMessageWriter _messageWriter;

    public DemoMiddleware(RequestDelegate next, IMessageWriter messageWriter)
    {
        _next = next;
        _messageWriter = messageWriter;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        await context.Response.WriteAsync(_messageWriter.GetMessage());
        await _next(context);
    }
}
```

This middleware simply injects a message from the IMessageWriter directly into the response
stream.

In order to register this, our users need to add 

```c#
services.AddSingleton<IMessageWriter, MessageWriter>();
```

to their `ConfigureServices` and

```c#
app.UseMiddleware<DemoMiddleware>();
```

to their `Configure`.

To make this easier to use for our users, we can leverage the `IStartupFilter` and
a convenient extension method.

```c#
public static class DemoMiddlewareRegistrationExtensions
{
    public static void AddDemoMiddleware(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddSingleton<IMessageWriter, MessageWriter>();
        services.AddTransient<IStartupFilter, DemoMiddlewareStartupFilter>();
    }
}
```

Here we're adding the IMessageWriter service and an IStartupFilter service. The IStartupFilter 
service looks like this:

{% highlight c# %}
public class DemoMiddlewareStartupFilter : IStartupFilter
{
    public Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next) =>
        app =>
        {
            app.UseMiddleware<DemoMiddleware>();
            next(app);
        };
}
{% endhighlight %}

Here we return an `Action` on `IApplicationBuilder` which is where we tell AspNetCore
to use our Middleware, remembering to call the passed in `next` afterwards.
_(Of course depending on our needs we may be terminating the call here, e.g. if we can't start up in a
consistent state we may not wish to continue starting up.)_

Now all our users need do is add one line to call this extension method from the `ConfigureServices` 
method of `Startup.cs`:

```c#
services.AddDemoMiddleware(Configuration);
```

_Note that I have passed the configuration through to the extension method purely for illustrative purposes
here. It's not used in this example and can be omitted if you don't need it in your code._

Now this is a contrived example, but you can see how we can use this approach to add sophisticated setup
logic to our APIs with a minimal amount of setup for the consumers of those APIs.

There is still a minor complaint with the approach as it stands. Our API users will expect
that only Service Registration occurs in the `ConfigureServices` method, but we're also
telling our Application to use this Middleware. That's a bit of a violation of the
Principle of Least Astonishment.

In general if we're going to add a whole suite of services and functionality, I think the
best place for that is on the `HostBuilder`.

If we add a new extension method to our `DemoMiddlewareRegistrationExtensions` as follows:

```c#
public static IWebHostBuilder UseDemoMiddleware(this IWebHostBuilder hostBuilder)
{
    hostBuilder.ConfigureServices((context, builder) => builder.AddDemoMiddleware(context.Configuration));
    return hostBuilder;
}
```

We can now simply amend our `CreateWebHostBuilder` method as follows:

```c#
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseDemoMiddleware();
```

Note that the GenericHost does not have the same `Startup` pattern as ASP.Net Core [(Instead it registers `IHostedService` instances)](https://github.com/aspnet/Hosting/issues/1163)
so the `IStartupFilter` can be registered, but will never be called.

In conclusion, `IStartupFilter` allows you to create a single point for consumers of your API to both
register and configure your services.
