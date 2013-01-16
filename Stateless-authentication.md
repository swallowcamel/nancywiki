This document provides an overview on how to enable stateless authentication in your Nancy application. Stateless authentication enables you to inspect each incoming request and, based on information about that request, decide if it should be treated as an authenticated request or not.

For instance you could inspect the request to make sure that a query string parameter was passed in (perhaps an apikey), that a certain header is available or that the request originated from a certain ip-address. The full request is at your disposal!

To enable stateless authentication, in your application, you need to complete the following steps

1. Install the `Nancy.Authentication.Stateless` package
1. Configure and enable Stateless Authentication
1. [Secure your modules](https://github.com/NancyFx/Nancy/wiki/Authentication-overview)

## Configure and enable stateless authentication

To enable Stateless Authentication, all you have to do it add a line like this to your bootstrapper:

```c#
StatelessAuthentication.Enable(pipelines, statelessAuthConfiguration);
```

The `statelessAuthConfiguration` variable, that is passed into `FormsAuthentication.Enable` method, is an instance of the `StatelessAuthenticationConfiguration` type, which enables you to customize the behavior of the stateless authentication provid

When creating an instance of the `StatelessAuthenticationConfiguration` type, it expects a single parameter of type `Func<NancyContext, IUserIdentity>`. The function is what is used to inspect the request (or anything else in the context for that matter) and return `null` if the request should not be treated as authenticated, or the appropriate [IUserIdentity](https://github.com/NancyFx/Nancy/wiki/Authentication-overview) if it should.

## Sample configuration

```c#
var configuration =
    new StatelessAuthenticationConfiguration(ctx =>
    {
        if (!ctx.Request.Query.apikey.HasValue)
        {
            return null;
        }

        // This would where you authenticated the request. IUserApiMapper is
        // not a Nancy type.
        var userValidator = 
            container.Resolve<IUserApiMapper>();

        return userValidator.GetUserFromAccessToken(ctx.Request.Query.apikey);
    });
```


[<< Part 20. The Super Simple View Engine](The Super Simple View Engine) - [Documentation overview](Documentation)