Besides defining handlers for specific routes, a module can also intercept requests that match one of its routes, both before and after the route is invoked. It is important to understand that these interceptors will only be invoked if the incoming request `matches one of the routes in the module`.

## Intercepting the request before the route is invoked

The Before interceptor enables you to intercept the request before it is passed to the appropriate route handler. This gives you a couple of possibilities such as modifying parts of the request or even prematurely aborting the request by returning a response that will be sent back to the caller.

    Before += ctx => {
        return <null or a Response object>;
    };

When defining a Before interceptor you use a slightly different syntax than you would do with a route handler. Since the interceptor will be invoked for all routes in the module, there is no need to define a pattern to match.

The parameter that is passed into the interceptor is an instance of the current NancyContext, from which you get access to things like context variables and the request.

The last difference is the return value of the interceptor. A return value of `null` means that no action is taken by the interceptor and that the request should process to be processed by the matching route. However, if the interceptor returns a `Response` of it’s own, the route will never be processed by the route and the response will be sent back to the client.

## Intercepting the request after the route was invoked

The After interceptor is defined using the same syntax as the Before interceptor, and the passed in parameter is also the current `NancyContext`. The difference is that the interceptor does not return a value. 

    After += ctx => {
        // Modify ctx.Response
    };

If you think about it for a moment, it makes sense. A `Before` interceptor is called before the route handler and it gives you the option to abort the request by sending back a response. With an After interceptor the route has already been handled and a response has been generated. Instead you can modify the existing response by accessing it on the context.

The interceptors are very useful when you want to perform tasks, per-request, on a module level for things like security, caching and rewriting requests and responses.