---
  title: Mapping complex types with AutoMapper
  header:
    subheading: A simple guide to Resolvers and Converters with AutoMapper
  categories:
    - csharp
  tags:
    - csharp
    - automapper
  permalink: /automapper/complex-mapping/
---


When you need to go beyond a simple mapping from a source member of one type to a destination member of another type, `AutoMapper` has some powerful tools in the shape of custom `Converters` and `Resolvers`. 
However, the documentation can be a little unclear as to which type to use for a given situation, and exactly how to register the type in your mapping configuration.
Furthermore, there are too many parameters supplied to the methods you need to implement and it can be confusing to write, much less come back and maintain later.

In this post, I will look at some common mapping scenarios and the `AutoMapper` tools to use in those scenarios. I will also suggest a style I use to make it clearer which parameters are useful.

I will assume that you are using `Dependency Injection` to configure/resolve `AutoMapper` and that you wish your custom `Converters` and `Resolvers` to be supplied by `Dependency Injection`. This is generally a "Good Idea"&trade; because if your `Converters` and `Resolvers` have any dependencies then they too will be resolved by DI. 

### `ITypeConverter:` Convert all instances of one Type to another
   
Imagine you have a custom `DateModel` that contains a Day, Month and Year. You know that you will always want this to be converted to a `DateTime`.

```csharp
    public class DateModel
    {
        public int Day { get; set; }
        public int Month { get; set; }
        public int Year { get; set; }
    }
```

In this scenario we use an `ITypeConverter<TSource, TDestination`. 

We tell it the *member* type we are mapping from and the *member* type we are mapping to and supplu a `Convert` method that tells us how to convert from the Source type to the Destination type.

```csharp
    public class DateConverter : ITypeConverter<DateModel, DateTime>
    {
        public DateTime Convert
        (
            DateModel source, 
            DateTime destination, 
            ResolutionContext context
        ) 
        => new DateTime
        (
            source.Year, 
            source.Month, 
            source.Day, 
            0, 0, 0, 
            DateTimeKind.Utc
        );
    }
```

A couple of things to note here:
1. We have access to the destination member.
1. We have access to the `ResolutionContext`.
    This can be used to pass data from the mapping configuration to the `Converter` and to gain access to the `Mapper` itself. I can't think of a good reason you would ever want to do that however, unless you want to give your future self a huge headache when trying to maintain the code!

I see very few use cases for the additional parameters, and, personally, I think this is a violation of the `Interface Segregation Principle` since I am forced to depend on some parameters I won't use. To make this clear, I usually borrow from the syntax for `discard parameters` and use underscores to show I will not be using the parameters. *(Note: the parameters are not actually discarded. I just use this style for clarity.)*

```csharp
    public class DateConverter : ITypeConverter<DateModel, DateTime>
    {
        public DateTime Convert
        (
            DateModel source, 
            DateTime _, ResolutionContext __
        ) 
        => new DateTime
        (
            source.Year, 
            source.Month, 
            source.Day, 
            0, 0, 0, 
            DateTimeKind.Utc
        );
    }
```

From within my Mapping Configuration I can map like so:

```csharp
CreateMap<DateModel, DateTime>()
    .ConvertUsing<DateConverter>();
```

Now every time I map from a SourceMember of type `DateModel` to `DateTime` my `DateConverter` will kick in and handle the mapping for me.

### `IValueConverter:` Convert instances of one Type to another for some Members only

An `ITypeConverter` is useful whwn you know that you will always map a given source type to a given destination type in one way only, but what if you want to choose how you map on a per-member basis. 

Imagine you have a `TelephoneNumbers` type that contains Mobile and Landline numbers:

```csharp
    public class TelephoneNumbers
    {
        public string Mobile { get; set; }

        public string Landline { get; set; }
    }
``` 

In this scenario we use an `IValueConverter<TSource, TDestination>`. 

As before, we tell it the *member* type we are mapping from and the *member* type we are mapping to and supply a `Convert` method that tells us how to convert from the Source type to the Destination type.

```csharp
public class MobileConverter : IValueConverter<TelephoneNumbers, string>
{
    public string Convert
    (
        TelephoneNumbers sourceMember, 
        ResolutionContext _
    )
    => sourceMember.Mobile;
}
```

Note that this time we don't get direct access to the destination member, and I have again used an underscore to make it clear to future readers of the code that I am not going to be using the `ResolutionContext`.

We now register this in our mapping configuration using `ConvertUsing<TValueConverter, TSourceMember>`, passing in an `Expression<Func<TSource, TSourceMember>>` to tell AutoMapper how to resolve the member we wish to map from.

```csharp
.ForMember
    ( 
        dest => dest.Mobile,
        opts => opts.ConvertUsing<MobileConverter, TelephoneNumbers>
                        (src => src.TelephoneNumbers)
    );
```

We need to specify the type of the Source Member because the MappingOptions only have access to the Source and Destination types and the type of the Destination member. The compiler is unable to infer the type from the `Expression`.

You can see how we could, using the above example, easily create a `LandlineConverter` for example.

### `IValueResolver<TSource, TDestination, TDestMember>` Access more than one of the source members when mapping to a destination member

Imagine you have a customer with a first and last name, but you want to map to a full name (and for some reason you can't change the source class).

```csharp
public class CustomerModel
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

public class Customer
{
    public string FullName { get; set; }
}
```

In this scenario we use an `IValueResolver<TSource, TDestination, TDestMember>`. We need to implement the `Resolve` method which takes a `TSource`, `TDestination`, `TDestMember` (as well as a `ResolutionContext`) and outputs a `TDestMember`.

I have never needed to access anything other than the `TSource`, so as before I adopt an "underscore syntax" so I can easily see which of the input parameters I am interested in:

```csharp
public class FullNameResolver 
    : IValueResolver<CustomerModel, Customer, string>
{
    public string Resolve
    (
        CustomerModel source, 
        Customer _, 
        string __, 
        ResolutionContext ___
    ) 
    => $"{source.FirstName} {source.LastName}";
}
```

This is now classed as a `Resolver` rather than a `Converter` (presumably because we are now accessing the full source object) and is registered using `MapFrom<TValueResolver>`:

```csharp
.ForMember
(
    dest => dest.FullName,
    opts => opts.MapFrom<FullNameResolver>()
);
```

### `IMemberValueResolver<in TSource, in TDestination, in TSourceMember, TDestMember>` Fully control mapping

If you really want to take over the mapping then the `IMemberValueResolver<TSource, TDestination, TSourceMember, TDestMember>` is available.

You have to implement a `TDestMember Resolve(TSource source, TDestination destination, TSourceMember sourceMember, TDestMember destMember, ResolutionContext context)`.

It's then registered in the same way as an `IValueResolver`.

The documentation suggests that you might use an `IMemberValueResolver` when you want to reuse a mapping from `TSourceMember` to `TDestMember` across types (by setting the `TSource` and `Tdest` to `object`), but in that situation I prefer to use an `IValueConverter`. Personally, I feel that if you have reached the stage that you need access to the source and destination objects as well as the source and destination members, then you have moved far beyound the use case of a mapper, but YMMV!

### LINQ Projections

On a final note, it is important to recognise that none of these custom `Converters` and `Resolvers` can be used in `.ProjectTo` operations. Only the `Expression` based operations can be converted to a SQL query by Automapper.
