*Note: This applies to Nancy 0.17.0 onwards, if you're still using a previous version of Nancy please refer to [[Legacy Routing]]*

Routes are defined in the constructor of a module. In order to define a route in Nancy, you need to specify a `Method` + `Pattern` + `Action` + (optional) `Condition`.

i.e
```c#
public class ProductsModule : NancyModule
{
    public ProductsModule()
    {
        Get["/products/{id}"] = _ =>
        {
            //do something
        };
    }
}
```

or async:

```c#
public class ProductsModule : NancyModule
{
    public ProductsModule()
    {
        Get["/products/{id}", runAsync: true] = async (_, token) =>
	{
	    //do something long and tedious
        };
    }
}
```


## Method

The Method is the [HTTP method](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) that is used to access the resource. Nancy supports the following methods `DELETE`, `GET`, `HEAD`, `OPTIONS`, `POST`, `PUT` and `PATCH`. Head requests are automatically handled for all routes that are declared for GET requests.

## Pattern

A route also needs a Pattern which declares the application-relative URL that the route answers to. The syntax of the Pattern is customizable but the default implementation that ships with Nancy supports capturing combinations of the following:

***Note:** The # in the parenthesis is the score*

1. **Literal segments** - (10,000) - `/some/literal/segments` which require an exact match.
2. **Capture segments** - (1,000) - `/{name}` which captures whatever is passed into the given segment of the requested URL and then passes it into the Action of the route. 
3. **Capture segments *(optional)*** - (1,000) - `/{name?}` by adding the question mark at the end of the segment name the segment can be made optional.
4. **Capture Segments *(optional/default)*** - (1,000) - `/{name?unnamed}/` by adding a value after the question mark we can turn an optional segment into a segment with a default value, should the pattern be the most suited but the segment missing, the value returned will be what comes after the question mark.
5. **RegEx Segment** - (1,000) - `/(?<age>[\d]{1,2})` using [Named Capture Grouped](http://www.regular-expressions.info/named.html) Regular Expressions, you can get a little more control out of the segment pattern. If you don't need to capture anything then you can use a non-capturing group like `(?:regex-goes-here)`
6. **Greedy Segment** - (0) - `/{name*}` by adding a star/asterisk to the end of the segment name, the pattern will match any value from the current forward slash onward. 
7. **Greedy RegEx Segment** - (100) - `^(?<name>[a-z]{3,10}(?:/{1})(?<action>[a-z]{5,10}))$` a segment composed of a RegEx and Greedy segment, this captures the entire path after the forward slash, The segment must start with `^` and end with `$` so we know the start/finish of the greedy regular expression segment.
7. **Multiple Captures Segment** - (100) - `/{file}.{extension}` or `/{file}.ext` a segment containing a mix of captures and literals.

Pattern segments can be combined, in any order, to create a complex Pattern for a route. It’s worth noting that capture segments are greedy, meaning they will match anything in the requested URL until another segment matches or until the end of the URL is reached.

## Pattern Scoring

Sometimes you may end up with two routes which end up giving a positive match.

For example say we defined the following:

```csharp
public class HomeModule : NancyModule
{
    public HomeModule()
    {
        Get["/{category}"] = _ =>
        {
            return "My category is " + _.category;
        }
    
        Get["/sayhello"] = _ =>
        {
            return "Hello from Nancy";
        }
    }
}
```

The first route contains a Captured Segment, while the second route is a Literal Segment.

When the routes are matched they get scored. 

A literal has a score of `10000` while a capture has a score of `1000`, if both routes contain the same number of segments then the route with the highest score wins and that route is picked.

If we browse to the URL `http://www.site.com/sayhello` both routes would match and the 2nd route would score `10000` and that method will execute.

## Action

A route Action is the behavior which is invoked when a request is matched to a route. It is represented by a lambda expression of type `Func<dynamic, dynamic>` where the dynamic input is a `DynamicDictionary`, a special dynamic type that is defined in Nancy and is covered in [[Taking a look at the DynamicDictionary]]

The response can be any model and the final result will be determined by [[Content Negotiation]]. However, if it is of the type `Response` then content negotiation will be ignored and the response will be sent straight back to the host.

The `Response` object declares several implicit cast operators which enables an action to also return, instead of a `Response` object, any of the following

1. `int` which will be interpreted as a HTTP status code of the response
2. `HttpStatusCode` enumerable value
3. `string` which will be interpreted as the body of the response
4. `Action<Stream>` which is a function that writes to the response stream

## Condition

The last part is an optional Condition that can be used to make sure that a route is only matched if certain conditions are met. This could e.g be a check to ensure that the route is only invoked if it was used by a mobile user-agent. A route condition is defined using a lambda expression of type `Func<NancyContext, bool>`.

```
Post["/login", (ctx) => ctx.Request.Form.remember] = _ => 
{
     return "Handling code when remember is true!";
}

Post["/login", (ctx) => !ctx.Request.Form.remember] = _ => 
{
     return "Handling code when remember is false!";
}
```

## Route Segment Constraints
*Note: Route segment constraints was introduced in version 0.21*

Route segment constraints allows you apply certain constraints to your captured route segments. To apply a constraint to a segment, simply add a `:` followed by the name of the constraint to the route segment:

```csharp
Get["/intConstraint/{value:int}"] = _ => "Value " + _.value + " is an integer.";
```

This will result in your route only getting hit if the value of the `value` segment is an integer.
  
The following constraints are available out of the box:

 - `int` - Allows only integer values.
 - `decimal` - Allows only decimal values.
 - `guid` - Allows only GUID values.
 - `bool` - Allows only boolean values.
 - `alpha` - Allows only values containing alphabetical character.
 - `datetime` - Allows only date values, optionally containing time.
 - `datetime(format)` - Allows only date and/or time values with the specified `format`. For format values, see [Custom Date and Time Format Strings](http://msdn.microsoft.com/en-us/library/8kb3ddd4.aspx)
 - `min(minimum)` - Allows only integer values with the specified `minimum` value.
 - `max(maximum)` - Allows only integer values with the specified `maximum` value.
 - `range(minimum, maximum)` - Allows only integer values within the specified range. (Between `minimum` and `maximum`)
 - `minlength(length)` - Allows only values longer than the specified minimum `length`.
 - `maxlength(length)` - Allows only values shorter that the maximum `length`.
 - `length(minimum, maximum)` - Allows only values with lengh within the specified range. (Between `minimum` and `maximum`)

### Custom Constraints

You can also implement your own, custom constraints. It's as easy as implementing `IRouteSegmentConstraint` and it will automatically be picked up by Nancy, SDHP-style. You can also derive your constraint from one of these convenience classes:

 - `RouteSegmentConstrainBase<T>` - Base class for a named constraint.
 - `ParameterizedRouteSegmentConstraintBase<T>` - Base class for a named constraint that accepts arguments.

#### Example

Here's an example implementation of an e-mail constraint:

```csharp
public class EmailRouteSegmentConstraint : RouteSegmentConstraintBase<string>
    {
        public override string Name
        {
            get { return "email"; }
        }

        protected override bool TryMatch(string constraint, string segment, out string matchedValue)
        {
            if (segment.Contains("@"))
            {
                matchedValue = segment;
                return true;
            }

            matchedValue = null;
            return false;
        }
    }
```

And usage:

```csharp
Get["/profile/{value:email}"] = _ => "Value " + _.value + " is an e-mail address.";
```

This route will only get hit as long as the `value` segment contains a `@`. The value that's passed to the route is the value returned through the `matchedValue` out parameter.

#### Resources

Head over to the [Constraints Sample Project](https://github.com/NancyFx/Nancy/tree/master/src/Nancy.Demo.ConstraintRouting) to look at some samples or take a look at the [existing constraint implementations](https://github.com/NancyFx/Nancy/tree/master/src/Nancy/Routing/Constraints).

## The secret for selecting the right route to invoke

There are a couple of gotchas you should be aware of when it comes to the default behavior, in Nancy, for selecting which route to invoke for a request. It sounds easy enough, you pick the one that matches the `Method`, `Pattern` and `Condition` of the request, right? In the simplest case that is true and how it is selected, but what if things are a bit more complicated?

You could, for example, have two routes with patterns that would capture the same request under certain circumstances. These two routes could either be defined in the same module or split across several modules. 

Turns out it’s not complicated after all, you just need to remember a couple of things

1. The order in which modules are loaded are non-deterministic in between application start-ups
2. Routes in a given module are discovered in the order in which they are defined
3. If there are several possible matches, the most specific match, i.e the route pattern that has the highest number of matching literal segments and fewest capture segments
4. If two, or more, routes are equal matches to a request, the first one is selected and which one this is depends on the module load order and the route order inside of the modules

## Going crazy with routes

Below are some samples of what routes in Nancy can look like. They cover some of the possible usages, but not all of them. 

```c#
// would capture routes like /hello/nancy sent as a GET request
Get["/hello/{name}"] = parameters => {
    return "Hello " + parameters.name;
};

// would capture routes like /hello/1234, but not /hello/asdf as a GET request
Get["/favoriteNumber/{value:int}"] = parameters => {
    return "So your favorite number is " + parameters.value + "?";
};

// would capture routes like /products/1034 sent as a DELETE request
Delete[@"/products/(?<id>[\d]{1,7})"] = parameters => {
    return 200;
};

// would capture routes like /users/192/add/moderator sent as a POST request
Post["/users/{id}/add/{category}"] = parameters => {
    return HttpStatusCode.OK;
};
```