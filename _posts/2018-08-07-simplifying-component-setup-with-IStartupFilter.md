---
  title: IStartupFilter
  header:
    subheading: Simplifying component setup with IStartupFilter
---

When registering services in ASP.Net Core, often you have to register 
the services in `Startup.ConfigureServices` and then tell the 
`IApplicationBuilder` to actually use them in `Startup.Configure`.

Take the example of JwtBearer Authentication. You have to remember to call 
```
services.AddAuthentication(options => ...)
```
in `ConfigureServices` _and_ 
```
app.UseAuthentication()
```
in `Configure`.

Not only is this open to making mistakes, it scatters the configuration of 
a service across multiple methods. This is not particularly convenient if
you want to distribute an API because consumers of your API don't have one 
place to register and configure your service.

Enter the `IStartupFilter`, part of the `Microsoft.AspNetCore.Hosting.Abstractions` 
package. It defines one method:
```
Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next);
```
_Note that although you can use an IStartupFilter with the Generic Host, it hasn't (at the time of writing) made into a non-AspNetCore package,
so you will need to refer to the AspNet version regardless of whether you are using AspNet or not._

When you call `Build` on your `HostBuilder` (either Generic or Web) the runtime retrieves an `IEnumerable` of `IStartupFilters` from its
`ServiceCollection` and then iterates through them in order invoking the `Configure` method on your app's `ApplicationBuilder`.

As a particularly contrived version, consider a middleware that has its own dependencies.
```
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
```services.AddSingleton<IMessageWriter, MessageWriter>();```
to their `ConfigureServices` and 
```app.UseMiddleware<DemoMiddleware>();```
to their `Configure`.

To make this easier to use for our users, we can leverage the `IStartupFilter` and
a convenient extension method. 

```
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
```
public class DemoMiddlewareStartupFilter : IStartupFilter
{
    public Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next) =>
        app =>
        {
            app.UseMiddleware<DemoMiddleware>();
            next(app);
        };
}
```
Here we return an `Action` on `IApplicationBuilder` which is where we tell AspNetCore
to use our Middleware, remembering to call the passed in `next` afterwards. 
_(Of course depending on our needs we may be terminating the call here, e.g. if our
Middleware detected something like an authentication error, or any other cause to
stop processing the request)_.

Now all our users need do is add one line to call this extension method from the `ConfigureServices` 
method of `Startup.cs`:     
```
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
```
public static IWebHostBuilder UseDemoMiddleware(this IWebHostBuilder hostBuilder)
{
    hostBuilder.ConfigureServices((context, builder) => builder.AddDemoMiddleware(context.Configuration));
    return hostBuilder;
}
```
We can now simply amend our `CreateWebHostBuilder` method as follows:
```
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseDemoMiddleware();
``` 
Remember, of course, that, at least at the time of writing, if you wish to use this with the GenericHost, 
you will need a separate extension method on IHostBuilder as well.

In conclusion, `IStartupFilter` allows you to create a single point for consumers of your API to both
register and configure your services.
