---
  title: More Assertions with Shouldly 
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

