The application pipelines enable you to perform tasks before and after routes are executed, and in the event of an error in any of the routes in the application. They behave the same way as the module pipelines but they are executed for all invoked routes, not just for the ones that are the module of the route that is being invoked.

## Intercepting the request before the route is invoked

The `Before` pipeline enables you to intercept the request before it is passed to the appropriate route handler. This gives you a couple of possibilities such as modifying parts of the request or even prematurely aborting the request by returning a response that will be sent back to the caller.

An application level `Before` hook is defined by a `Func<NancyContext, Response>` function and is used in the following way

```c#
pipelines.BeforeRequest += (ctx) => {
    return <null or a Response object>;
};
```

The parameter that is passed into the pipeline is an instance of the current `NancyContext`, from which you get access to things like context variables and the request.

A return value of `null` means that no action is taken by the hook and that the request should be processed by the matching route. However, if the interceptor returns a `Response` of its own, the request will never be processed by a route and the response will be sent back to the client.

## Intercepting the request after the route was invoked

The `After` pipeline is defined using the same syntax as the `Before` pipeline, and the passed in parameter is also the current `NancyContext`. The difference is that the hook does not return a value.

An application level `After` hook is defined by a `Action<NancyContext>` function and is used in the following way

```c#
pipelines.AfterRequest += (ctx) => {
    // Modify ctx.Response
};
```

The `After` hooks does not have any return value because one has already been produced by the appropriate route. Instead you get the option to modify (or completely replace) the existing response by accessing the `Response` property of the `NancyContext` that is passed in.

## Intercepting the request when an error occurred

The `OnError` interceptor enables you to execute code whenever an exception occurrs in any of the routes that are being invoked. It gives you access to the `NancyContext` and the exception that took place.

An application level `OnError` hook is defined by a `Func<NancyContext, Exception, Response>` function and is used in the following way

```c#
pipelines.OnError += (ctx, ex) => {
    return null;
};
```

**A note on System.AggregateExceptions on the OnError pipeline**

The route execution is handled by a number of nested Tasks (`System.Threading.Tasks.Task`). If anything goes wrong when those Tasks are being invoked, the exceptions are wrapped in a nested hierarchy of `System.AggregateException`. `System.AggregateException` can hold as many exceptions as are thrown. While this is for most cases only one exception, we could potentially see multiple exceptions.

If only one exception is wrapped, then Nancy will unwrap that exception and hand it to the `OnError` pipeline. If a collection of exceptions is wrapped, then Nancy will pass on a flattened `System.AggregateException` in order not to swallow any exceptions.

##Wiring up your hooks

To create application level hooks you define them in your [Bootstrapper](Bootstrapper). They can be defined either in the `ApplicationStartup` or `RequestStartup` methods. This is because you might need to use something from the container in your hook, and the different methods let you resolve from the right container depending on your scoping requirements.

```c#
protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines)
{
}

protected override void RequestStartup(TinyIoCContainer requestContainer, IPipelines pipelines, NancyContext context)
{
}
```

The hooks are created by accessing the appropriate property on the `pipelines` parameter. The pipelines parameter is of the type `IPipelines` and gives you access to the `BeforeRequest`, `AfterRequest` and `OnError` properties.

[<< Part 14. Adding a custom favicon](Adding a custom favicon) - [Documentation overview](Documentation) - [Part 16. Authentication overview >>](Authentication overview)