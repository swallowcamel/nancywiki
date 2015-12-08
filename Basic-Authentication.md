This document provides an overview on how to enable basic authentication in your Nancy application. For a full working sample, please refer to the `Nancy.Demo.Authentication.Basic` project in the Nancy solution.

To enable forms authentication in your application, you need to complete the following steps

1. Install the `Nancy.Authentication.Basic` package
1. Implement an `IUserMapper`
1. Configure and enable Basic Authentication
1. Securing your modules/routes

## The user mapper

The user mapper is responsible for mapping a user from an identifier. The identifier is the token that will be put in the authentication cookie which will be used to re-establish the identity of the user that is performing the request, so that you do not need to enter your credentials for each request.

We have chosen to use a GUID as the identifier. The reason for this is that using something like the username or id is a potential vulnerability, because, if the cookie encryption was somehow compromised,  it would be easy to spoof the identity of another user by guessing their username or id.

The identifier should never be read by a human, so making it a GUID also discourages that it is ever displayed anywhere, and each GUID is both unique and non-sequential, making it a lot harder to spoof than the identity of someone else.

It is also important to know that the identifier should be treated as permanent for the user that it was generated for and will be reused across requests and application sessions. 

The `IUserMapper` interface is defined in the following way

```c#
public interface IUserMapper
{
    /// <summary>
    /// Get the real username from an identifier
    /// </summary>
    /// <param name="identifier">User identifier</param>
    /// <param name="context">The current NancyFx context</param>
    /// <returns>Matching populated IUserIdentity object, or empty</returns>
    IUserIdentity GetUserFromIdentifier(Guid identifier, NancyContext context);
}
```

A single method whose sole responsibility is to take the GUID identifier and return the corresponding user. If the identifier could not be mapped to a user then `null` should be returned. For more information about the `IUserIdentity` interface, refer to the [Authentication overview](Authentication overview).

## Modifying your application to handle forms authentication

Once you have the required package installed, it is time to enable basic authentication, but hooking it up to the [application pipeline](The Application Before, After and OnError pipelines) in your [bootstrapper](Bootstrapper).

This is done by calling the `EnableBasicAuthentication` extension method (which is installed with the `Nancy.Authentication.Basic` package), on the `IPipelines`.

```c#
public class AuthenticationBootstrapper : DefaultNancyBootstrapper
{
    protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines)
    {
        base.ApplicationStartup(container, pipelines);

        pipelines.EnableBasicAuthentication(new BasicAuthenticationConfiguration(                   
            container.Resolve<IUserValidator>(),
            "MyRealm"));
    }
}
```

The `EnableBasicAuthentication` extension method expects an instance of the `BasicAuthenticationConfiguration` class, which has the following values

- An `IUserValidator` implementation (see above) that is used to resolve the user based on the passed credentials
- A string representing the name of the authentication `realm``
- A `UserPromptBehaviour` enum value to control when the browser should prompt the user for credentials, the default value is `NonAjax` (see below for full list of available options)

## The UserPromptBehaviour enum

To control when the user should be prompted for credentials. One of the following values are valid

|Name|Description|
|----|-----------|
|Never|Never present user with login prompt|
|Always|Always present user with login prompt|
|NonAjax|Only prompt the user for credentials on non-ajax requests|

## Securing your routes / modules

Once everything is wired up, it is time to secure your modules or individual routes. For a more indepth overview, please read [securing your resources](Authentication-overview#securing-your-resources)