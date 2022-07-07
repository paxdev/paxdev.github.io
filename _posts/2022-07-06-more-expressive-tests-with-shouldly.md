---
  title: More Expressive Tests with Shouldly 
  tags:
    - csharp
    - tdd
---

There are a couple of key things all good Unit Tests should do:

1. Tell you clearly _what_ they're testing.
1. Tell you clearly _why_ they fail.

Built-in assertion libraries don't really help us to meet these two expectations. Take a simple boolean assertion in `xUnit`:

```
var response = new { IsSuccessStatusCode = false };

Assert.True(response.IsSuccessStatusCode);

/*
Assert.True() Failure
Expected: True
Actual:   False
*/
```

The Assert syntax is not very easy to read and the failure message doesn't tell us _what_ was being tested. 

Things get worse when we need to compare values - see if you can spot the deliberate mistake.

```
var result = new { StatusCode = 404 };

Assert.Equal(result.Value, 200);

/*
Assert.Equal() Failure
Expected: 404
Actual:   200
*/
```

Not only do I not know _what_ was being tested, but due to a specific syntax I've (deliberately!) mixed up the _expected_ and _actual_ values - I actually expected `response.StatusCode` to equal 200 but it's trivially simple to get the values the wrong way round.
In my experience as a developer, if you give someone the chance to make a mistake, they _will_ make it!

`nUnit` makes things _slightly_ better:

```
var response = new { StatusCode = 404 };

Assert.That(response.StatusCode, Is.Not.EqualTo(404));

/*
Expected: not equal to 404
But was:  404
*/
```

It's easier to get the _actual_ and _expected_ the right way round and both the code and the failure message are slightly easier to read, but it still doesn't tell me _what_ it expected not to equal 404 and, honestly, who says "Assert.That ..." in everyday speech - and that's a _lot_ of typing.

`Shouldly` (https://shouldly.io/) is a library that helps to solve all these issues (and more.)

We can add from the command line (or use Manage Packages in Visual Studio)

```
dotnet add package Shouldly
```

Our assertions now become:

```
var response = new { IsSuccessStatusCode = false };

response.IsSuccessStatusCode.ShouldBeTrue();

/*
response.IsSuccessStatusCode
    should be
True
    but was
False
*/

var response = new { StatusCode = 404 }; 

response.StatusCode.ShouldNotBe(404);

/*
response.StatusCode
    should not be
404
    but was
*/
```

Not only is it immediately clear from my tests what I'm testing and what I expect my results to be but the test failure messages tell me what is being tested, what results were expected _and_ what the actual results were.
In addition the syntax is much simpler to get right and it follows natural speech patterns so I don't have to spend valuable time deciphering C# syntax when reading/debugging code.

There are additional advantages. 

As you might expect, exceptions are handled (excuse the pun) gracefully:

```
class Throws
{
    public void GetResult() => throw new AccessViolationException();
}

var sut = new Throws();

Should.Throw<BadImageFormatException>(sut.GetResult);

/*
`sut.GetResult`
    should throw
System.BadImageFormatException
    but threw
System.AccessViolationException
*/
```

The `Should.Throw` method returns the thrown `Exception` so you can continue to test the message or any other properties of the `Exception` you wish.

As with any Assertion library there are useful assertions for `Types`, `Collections`, `Strings` and so forth, however there is one final killer feature which makes `Shouldly` stand out in my opinion.

Whilst in a `Unit Test` you should strive to only have one logical assertion per test, when you have something like a `System Integration Test` which has a lot of complex and expensive setup, you may wish to run a number of tests from a given setup, e.g. was the database updated correctly, did any logs get created, were APIs called correctly as well as checking the format and state of the returned HttpResponse.

The problem with a standard Assertion Library is that the first failed Assertion will cause the test to stop running and you won't get feedback from any further tests.

`Shouldly` solves this problem with `ShouldSatisfyAllConditions`:

```
var response = new { StatusCode = 503 };
var repository = new {RecordCount = 7};
var logs = new {Entries = new[] {new {Message = "Failed"}}};

response.ShouldSatisfyAllConditions
(
    () => response.StatusCode.ShouldBe(200),
    () => repository.RecordCount.ShouldBe(2),
    () => logs.Entries.ShouldContain(e => e.Message == "Success")
);

/*
response
    should satisfy all the conditions specified, but does not.
The following errors were found ...
--------------- Error 1 ---------------
    response.StatusCode
        should be
    200
        but was
    503

--------------- Error 2 ---------------
    repository.RecordCount
        should be
    2
        but was
    7

--------------- Error 3 ---------------
    logs.Entries
        should contain an element satisfying the condition
    (e.Message == "Success")
        but does not

-----------------------------------------
*/
```

So you've got great feedback for _all_ the failed assertions in this test and not just the first.

In this post I've given a very quick introduction to `Shouldly's` capabilities. I hope it helps you in your work.

Happy coding!