---
  title: MSpec Fakes with Async Await
  header:
    subheading: Reducing boilerplate code and clarifying intent
  categories:
    - csharp
  tags:
    - csharp
    - tdd
    - mspec
---

In general, the C# language gives us lots of simple constructs to write asynchronous code simply and elegantly, but unfortunately when we come to write unit tests we can find ourselves having to write ugly, unintuitive boilerplate code in order to get our async code to run in MSpec Fakes.

Consider the following System Under Test:

```csharp
public class TestSubject
{
    readonly IAsyncDependency _asyncDependency;

    public TestSubject(IAsyncDependency asyncDependency)
    {
        _asyncDependency = asyncDependency;
    }

    public async Task<ValidationResult> CallDependency()
    {
        return await _asyncDependency.IsValid();
    }
}

public interface IAsyncDependency
{
    Task<ValidationResult> IsValid();
}

public abstract class TestResult { }

public class Valid : TestResult { }

public class Invalid : TestResult { }
```

We have a `TestSubject` class with a dependency on an `async` method that returns a `Task<ValidationResult>`. We want to test what happens when the dependency returns an instance of `Valid`, so we start to write the following code to set up behaviour on our mock:

```csharp
public class When_it_is_valid : WithSubject<TestSubject>
{
    Establish context = () => 
        The<IAsyncDependency>()
            .WhenToldTo(d => d.IsValid())
            .Return(new Valid());

    ...
}
```

However the compiler will complain that it cannot convert from `Valid` to `Task<TestResult>`. So, we can try to return a `Task` using `Task.FromResult`.

```csharp
    Establish context = () => 
        The<IAsyncDependency>()
            .WhenToldTo(d => d.IsValid())
            .Return(Task.FromResult(new Valid()));
```

However the compiler is _still_ complaining because now we are returning a `Task<Valid>`, not a `Task<TestResult>`. We now need to add a cast to our behaviour setup:

```csharp
    Establish context = () => 
        The<IAsyncDependency>()
            .WhenToldTo(d => d.IsValid())
            .Return(Task.FromResult((TestResult)new Valid()));
```

... and all of a sudden we have managed to bury the intent of our test (to assert behaviour when a `Valid` is returned) under a mass of boilerplate code that is there simply to setup a mock to return a `Task`.

Ideally we would like something similar to Moq's `.ReturnsAsync` setup syntax which clearly expresses our intention whilst saving us all that nasty boilerplate.

If we look at the code behind MSpec's WhenToldTo, we find that it takes an Expression that it uses the expression we wish to perform a setup on and returns an `IQueryOptions<TReturnValue>`.

```csharp
    public static IQueryOptions<TReturnValue> WhenToldTo<TFake, TReturnValue>(
        this TFake fake,
        Expression<Func<TFake, TReturnValue>> func) 
```

Looking at the methods on `IQueryOptions<TReturn>` we find that we will need to supply either a return value of, or a function that returns a type of `Task`: 

```csharp
  public interface IQueryOptions<TReturn>
  {
    void Return(TReturn returnValue);

    void Return(Func<TReturn> valueFunction);

    void Return<T>(Func<T, TReturn> valueFunction);

    ...
  }
```

We can easily accomplish this by creating a set of extension methods that wrap our return value, or return value function in a `Task<T>`:

```csharp
public static class ReturnAsyncExtensions
{
    public static void ReturnAsync<TReturn>(this IQueryOptions<Task<TReturn>> queryOptions, TReturn returnValue)
        => queryOptions.Return(Task.FromResult(returnValue));

    public static void ReturnAsync<TReturn>(this IQueryOptions<Task<TReturn>> queryOptions, Func<TReturn> valueFunction)
        => queryOptions.Return(() => Task.FromResult(valueFunction()));

    public static void ReturnAsync<TReturn, T>(this IQueryOptions<Task<TReturn>> queryOptions,
                                                Func<T, TReturn> valueFunction)
        => queryOptions.Return((T arg) => Task.FromResult(valueFunction(arg)));

    ...
}
```

An interesting consequence of this approach is that the compiler can now infer the correct return type from the method we are setting up the behaviour for, so we no longer need any casting from subclasses, and finally we can write simple code that clearly expresses our intent:

```csharp
    Establish context = () => 
        The<IAsyncDependency>()
            .WhenToldTo(d => d.IsValid())
            .ReturnAsync(new Valid());
```

Obviously, this will not help us if we want to test more sophisticated behaviour such as passing `CancellationToken` or `IProgress`, but in those cases, we are likely to need bespoke setup for each subject under test anyway.

The full source code for this is on [GitHub](https://github.com/paxdev/PaxDev.Machines.Fakes.Async) or can be [downloaded from NuGet](https://www.nuget.org/packages/PaxDev.Machine.Fakes.Async/) (Package Name: `PaxDev.Machine.Fakes.Async`)
