Routes are defined in the constructor of a module. In order to define a route in Nancy, you need to specify a `Method` + `Pattern` + `Action` + (optional) `Condition`.

##Method

The Method is the [HTTP method](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) that is used to access the resource. Nancy supports the following methods `DELETE`, `GET`, `HEAD`, `OPTIONS`, `POST`, `PUT` and `PATCH`. Head requests are automatically handled for all routes that are declared for GET requests.

##Pattern

A route also needs a Pattern which declares the application-relative URL that the route answers to. The syntax of the Pattern is customizable but the default implementation that ships with Nancy supports capturing combinations of the following

1. Literal segments `/some/literal/segments` which require an exact match
2. Capture segments `/{name}` which captures whatever is passed into the given segment of the requested URL and then passes it into the Action of the route.  Capture segments must use only alphanumeric characters.
3. Regular expression segments `/(?<age>[\d]{1,2})` which uses [named capture groups](http://www.regular-expressions.info/named.html) to provide the same behavior as capture segments, but with much greater control over what will be matched

The three kinds of pattern segments can be combined, in any order, to create a complex Pattern for a route. It’s worth noting that capture segments are greedy, meaning they will match anything in the requested URL until another segment matches or until the end of the URL is reached.

## Action

A route Action is the behavior which is invoked when a request is matched to a route. It is represented by a lambda expression of type `Func<dynamic, Response>` where the dynamic input is a `DynamicDictionary`, a special dynamic type that is defined in Nancy and is covered in [[Taking a look at the DynamicDictionary]]

The response part of the lambda is a Nancy Response object, which enables you to define things like headers and response body. 

The Response object declares several implicit cast operators which enables an action to also return, instead of a Response object, any of the following

1. `int` which will be interpreted as a HTTP status code of the response
2. `HttpStatusCode` enumerable value
3. `string` which will be interpreted as the body of the response
4. `Action<Stream>` which is a function that writes to the response stream

## Condition

The last part is an optional Condition that can be used to make sure that a route is only matched if certain conditions are met. This could e.g be a check to ensure that the route is only invoked if it was used by a mobile user-agent. A route condition is defined using a lambda expression of type `Func<bool>`.

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
        return HttpStatusCode.Ok;
    };

[<< Part 2. Exploring the Nancy module](Exploring the Nancy module) - [Documentation overview](Documentation) - [Part 4. Taking a look at the DynamicDictionary >>](Taking a look at the DynamicDictionary)