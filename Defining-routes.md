*Note: This applies to Nancy 0.17.0 onwards, if you're still using a previous version of Nancy please refer to [[Legacy Routing]]*

Routes are defined in the constructor of a module. In order to define a route in Nancy, you need to specify a `Method` + `Pattern` + `Action` + (optional) `Condition`.

i.e

	public class ProductsModule : NancyModule
	{
		public ProductsModule
		{
			Get["/products/{id}"] = _ =>
			{
		  		//do something
			};
		}
	}


## Method

The Method is the [HTTP method](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) that is used to access the resource. Nancy supports the following methods `DELETE`, `GET`, `HEAD`, `OPTIONS`, `POST`, `PUT` and `PATCH`. Head requests are automatically handled for all routes that are declared for GET requests.

## Pattern

A route also needs a Pattern which declares the application-relative URL that the route answers to. The syntax of the Pattern is customizable but the default implementation that ships with Nancy supports capturing combinations of the following

1. **Literal segments** - `/some/literal/segments` which require an exact match.
2. **Capture segments** - `/{name}` which captures whatever is passed into the given segment of the requested URL and then passes it into the Action of the route. 
3. **Capture segments *(optional)*** - `/{name?}` by adding the question mark at the end of the segment name the segment can be made optional.
4. **Capture Segments *(optional/default)*** - `/{name?unnamed}/` by adding a value after the question mark we can turn an optional segment into a segment with a default value, should the pattern be the most suited but the segment missing, the value returned will be what comes after the question mark.
5. **RegEx Segment** - `/(?<age>[\d]{1,2})` using [Named Capture Grouped](http://www.regular-expressions.info/named.html) Regular Expressions, you can get a little more control out of the segment pattern. If you don't need to capture anything then you can use a non-capturing group like `(?:regex-goes-here)`
6. **Greedy Segment** - `/{name*}` by adding a star/asterisk to the end of the segment name, the pattern will match any value from the current forward slash onward. 
7. **Greedy RegEx Segment** - `^(?<name>[a-z]{3,10}(?:/{1})(?<action>[a-z]{5,10}))$` a segment composed of a RegEx and Greedy segment, this captures the entire path after the forward slash, The segment must start with `^` and end with `$` so we know the start/finish of the greedy regular expression segment.

Pattern segments can be combined, in any order, to create a complex Pattern for a route. It’s worth noting that capture segments are greedy, meaning they will match anything in the requested URL until another segment matches or until the end of the URL is reached.

## Pattern Scoring

*to explain*...

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