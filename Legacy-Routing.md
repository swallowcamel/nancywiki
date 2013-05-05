Routes are defined in the constructor of a module. In order to define a route in Nancy, you need to specify a `Method` + `Pattern` + `Action` + (optional) `Condition`.

## Method

The Method is the [HTTP method](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) that is used to access the resource. Nancy supports the following methods `DELETE`, `GET`, `HEAD`, `OPTIONS`, `POST`, `PUT` and `PATCH`. Head requests are automatically handled for all routes that are declared for GET requests.

## Pattern

A route also needs a Pattern which declares the application-relative URL that the route answers to. The syntax of the Pattern is customizable but the default implementation that ships with Nancy supports capturing combinations of the following

1. Literal segments `/some/literal/segments` which require an exact match
2. Capture segments `/{name}` which captures whatever is passed into the given segment of the requested URL and then passes it into the Action of the route. Capture segments can be made optional by adding a question mark at the end of the name `/{name?}`. You can also provide a default value for optional segments, which will be used when the segment could not be matched `/{name?Nancy}`. Furthermore a capture segment can be made greedy by adding an asterisk at the end of the name `/{name*}`. This will catch any input including slashes(/) until the next literal segment. 
3. Regular expression segments `/(?<age>[\d]{1,2})` which uses [named capture groups](http://www.regular-expressions.info/named.html) to provide the same behavior as capture segments, but with much greater control over what will be matched. Regex segments needs a capturing group to be identified. If you do not need to capture anything, then you can use a non-capturing group `(?:regex-goes-here)` 

The three kinds of pattern segments can be combined, in any order, to create a complex Pattern for a route. It’s worth noting that capture segments are greedy, meaning they will match anything in the requested URL until another segment matches or until the end of the URL is reached.

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

// would capture routes like /products/1034 sent as a DELETE request
Delete[@"/products/(?<id>[\d]{1,7})"] = parameters => {
    return 200;
};

// would capture routes like /users/192/add/moderator sent as a POST request
Post["/users/{id}/add/{category}"] = parameters => {
    return HttpStatusCode.OK;
};
```

## Defining your own route pattern syntax

If you ever find yourself in a place where the default capture functionality does not cover your requirements, or if you just do not like the syntax, then you are in luck! For instance if you wanted your routes to be defined as `/users/@name/:id`, where @ signaled that it has to only contain letters, and : signaled that it had to be numerical, you could do that very easily.

### The IRoutePatternMatcher interface

This interface is responsible for telling Nancy if the incoming request matches the route that is passed in. By providing your own implementation, you can support what ever syntax and/or functionality that you wish.

```c#
/// <summary>
/// Defined the functionality that is required by a route pattern matcher.
/// </summary>
/// <remarks>Implement this interface if you want to support a custom route syntax.</remarks>
public interface IRoutePatternMatcher
{
    /// <summary>
    /// Attempts to match a requested path with a route pattern.
    /// </summary>
    /// <param name="requestedPath">The path that was requested.</param>
    /// <param name="routePath">The route pattern that the requested path should be attempted to be matched with.</param>
    /// <param name="segments"> </param>
    /// <param name="context">The <see cref="NancyContext"/> instance for the current request.</param>
    /// <returns>An <see cref="IRoutePatternMatchResult"/> instance, containing the outcome of the match.</returns>
    IRoutePatternMatchResult Match(string requestedPath, string routePath, IEnumerable<string> segments, NancyContext context);
}
```

The following parameters are provided

- `requestedPath` - The application relative path of the incoming request, for example `/users/123`
- `routePath` - The pattern of the route that Nancy wants to know if the request matches, for example `/users/{id}`
- `segments` - A split version of the `routePath` value. More about this below
- `context` - An instance of the `NancyContext` type, for the incoming request

The `Match` method is expected to return a [RoutePatternMatchResult](https://github.com/NancyFx/Nancy/blob/master/src/Nancy/Routing/RoutePatternMatchResult.cs), which indicates if there was a match, or not, and the parameters that were captured.

### The IRouteSegmentExtractor interface

This interface is responsible for breaking a route pattern into parts which can then be passed in to the `IRoutePatternMatcher`. This can simplify working with the route when determining if it is a match.

```c#
/// <summary>
/// Defines the functionality for extracting the individual segments from a route path.
/// </summary>
public interface IRouteSegmentExtractor
{
    /// <summary>
    /// Extracts the segments from the <paramref name="path"/>;
    /// </summary>
    /// <param name="path">The path that the segments should be extracted from.</param>
    /// <returns>An <see cref="IEnumerable{T}"/>, containing the extracted segments.</returns>
    IEnumerable<string> Extract(string path);
}
```

What a segment means can vary between implementations. For the default implementation, the `DefaultRouteSegmentExtractor`, it will split the route pattern on every occurrence of a forward slash and return the parts as the segments, so `/foo/{bar}/(?<id>[\d]*)` would be split up into `foo`, `{bar}` and `(?<id>[\d]*)`.

### Wiring it all up

Once you have implemented your own route pattern matcher, the last thing you need to do is to tell Nancy to use it. You do this by overriding the `NancyInternalConfiguration` property, on your [[Bootstrapper]] and point the `RoutePatternMatcher` to your implementation.

```c#
public class Bootstrapper : DefaultNancyBootstrapper
{
    protected virtual NancyInternalConfiguration InternalConfiguration
    {
        get
        {
            return NancyInternalConfiguration.WithOverrides(x => x.RoutePatternMatcher = typeof(MyRoutePatternMatcher));
        }
    }
}
```

If you are replacing the route segment extractor, then you do the same thing except you provide a new value of the `RouteSegmentExtractor` property on the `NancyInternalConfiguration`.