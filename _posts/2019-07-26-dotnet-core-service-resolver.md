---
  title: A Service Resolver for .Net Core
  header:
    subheading: Using Dependency Injection outside of a website or service
  tags:
    - dotnetcore
    - csharp
---

The ASP.Net Core framework comes with great built-in support for Dependency Injection and IoC, which has recently been extracted out into the `IHostBuilder` interface allowing you to use the built-in IoC outside of an ASP.Net Core web application.

The problem is that this approach is best suited to a long-running service that responds to some sort of message and is not best suited to a synchronous process that starts up, runs for a determined period and then shuts down, for example an Amazon Lambda Function that wakes up, processes some files in S3 and then terminates.

In the past, people might have attempted to solve this problem with a ServiceLocator, but this is no longer considered good practice for a number of reasons:

* If we have a lot of `ServiceLocator.Resolve<Type>()` calls all over our code, we may as well simply instantiate instances of our classes all over the place.
* We lose the ability to scope our services, e.g. in our Amazon Lamdba Function maintaining a Database Transaction for each file processed.
* We have to be very careful in managing the lifecycle and disposal of the dependencies we create - for example HTTP connections.

What is needed is a `ServiceProvider` that can resolve services within a scope, managing the lifecycles accordingly.

To this end I have created
