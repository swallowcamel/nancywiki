Explaining the authentication support in Nancy is pretty easy - there really is not any. That is not to say that you cannot do authentication with Nancy, or that it is not on the “super duper happy path”, but at the core Nancy contains only some very basic constructs around authentication, which other packages (or your own) can take advantage of to provide many types of authentication.

Instead of shipping inflexible, hard coded, “built in” authentication providers, the shipped authentication facilities takes advantage of extension points in Nancy, such as Application pipelines, Module pipelines, `NancyContext` and normal extension methods to provide the desired functionality.

What this means is that if the default implementations we ship in the box do not suit your business needs, you can simply write your own without having to worry about conforming to a complex set of predefined interfaces, base classes, or other constraints that a hard coded authentication awareness would impose on you.

Out-of-the-box we ship the following authentication mechanisms, each of which is activated by installing the corresponding (name in the parenthesis) Nuget package.

 - Forms (`Nancy.Authentication.Forms`)
 - Basic (`Nancy.Authentication.Basic`)
 - Stateless (`Nancy.Authentication.Stateless`)

## What did you say your name was again?

Even though Nancy does not know anything about authenticating a user, it still understands the concept of a user. In Nancy a user is defined by the `IUserIdentity` interface, which provides the basic information about a user's identity

```c#
public interface IUserIdentity
{
    /// <summary>
    /// Gets or sets the name of the current user.
    /// </summary>
    string UserName { get; set; }

    /// <summary>
    /// Gets or set the claims of the current user.
    /// </summary>
    IEnumerable<string> Claims { get; set; } 
}
```

This interface should be implemented by you to provide a domain specific user object that contains information relevant to your application.

To gain access to the current authenticated user, you simply access the `CurrentUser` property on the `NancyContext`. A value of `null` means that the user of the current request has not been authenticated, anything else means the user has been authenticated.

The context should be available in most places of Nancy so it should never be an issue to get the identity of the user that is invoking the current request.

## Securing your resources

Establishing the identity of the current user is only half of the equation. You also need to have a way to secure resources in your application, so that they are only permitted to be accessed by authenticated users and perhaps even users that meet certain claims.

Resources are secured at a module or route level (of course, you could secure all resources by creating an [application pipeline](The Application Before, After and OnError pipelines) if you have the need) by inspecting the `NancyContext.CurrentUser` property and making sure that it is not null.

This task can be performed in a `Before` hook on the [module pipeline](The before and after module hooks), which ensures it executed before any route in the module is invoked. The hook also gives us the ability to prematurely terminate the execution of the request and return another response, as is required when an unauthenticated user is trying to access a secure resource.

```c#
public class SecureModule : NancyModule
{
    public SecureModule()
    {
        Before += ctx => {
            return (this.Context.CurrentUser == null) ? new HtmlResponse(HttpStatusCode.Unauthorized) : null;
        };
        
        // Your routes here
    }
}
```

Optionally you would also verify the claims of the user, but that is the basic gist of how to secure resources in a module. 

Adding that code to each of your secure modules is a violation of the DRY principle and is, quite frankly, a tedious task. Enter extension methods!

Nancy comes with a couple of extension methods that encapsulate these tasks and drastically cut down the amount of code you have to write.

The following extension methods are available

- `RequiresAuthentication` - Ensures that an authenticated user is available or it will return `HttpStatusCode.Unauthorized`. The `CurrentUser` must not be `null` and the `UserName` must not be empty for the user to be considered authenticated.
- `RequiresClaims` - Enables you to define the list of claims that the user has to meet in order to be granted access to the secured resources. All listed claims have to be met for access to be granted
- `RequiresAnyClaim` - Same as `RequiresClaims` but with the difference that at least one of the listed claims has to be met in order for access to be granted
- `RequiresValidatedClaims` - Gives you full control over the claims validation process, by letting you define a function, in the format `Func<IEnumerable<string>, bool>` that is invoked to determine if access should be granted or not
- `RequiresHttps` - While this is not strictly related to authentication, it tells Nancy that the resources are only permitted to be accessed if the request uses https. There is an optional bool parameter that can be used to control if the user should be redirected to the resources, as a https request, if an attempt is made to access it using http. This can be used to ensure that resources are always accessed using a secure request

All of these are extension methods on the `NancyModule` class, which makes it trivial to add your own or to mimic the behaviors on an application level. You need to include the types in the `Nancy.Security` namespace in order for these extension methods to be available from inside your module.

Using these extension methods, the previous example can be updated to look like the following code

```c#
public class SecureModule : NancyModule
{
    public SecureModule()
    {
        this.RequiresAuthentication();
    }
        
    // Your routes here
}
```

Of course they can be combined as well

```c#
public class SecureModule : NancyModule
{
    public SecureModule()
    {
        this.RequiresHttps();
        this.RequiresAuthentication();
        this.RequiresClaims(new [] { "Admin" });
    }
        
    // Your routes here
}
```

All of these can also be used on a per-route level

```c#
public class SecureModule : NancyModule
{
   public SecureRoute()
   {
      Get["/"] = _ =>
      {
         this.RequiresAuthentiction();
         return 200;
      };
   }
}
```

To be granted permission to the routes in the `SecureModule` above, the user has to be accessing it using https, be authenticated, and have the Admin claim. If either of these are untrue then permissions will be denied.

## Creating your own security extensions

To create your own security extensions you simply add extension methods to the `NancyModule` that attaches to the `Before` pipeline and inspects the credentials. 

For example, this is how `RequiresAuthentication` works

```c#
public static class ModuleSecurity
{
    public static void RequiresAuthentication(this NancyModule module)
    {
        module.Before.AddItemToEndOfPipeline(RequiresAuthentication);
    }

    private static Response RequiresAuthentication(NancyContext context)
    {
        Response response = null;
        if ((context.CurrentUser == null) ||
            String.IsNullOrWhiteSpace(context.CurrentUser.UserName))
        {
            response = new Response { StatusCode = HttpStatusCode.Unauthorized };
        }

        return response;
    }

}
```
This makes sure that the `CurrentUser` has been set and that it has a valid `UserName` in which case it will return `null`, meaning that the execution should continue as normal. Invalid credentials will result in a response, with the `HttpStatusCode.Unauthorized` status code, to be returned which will terminate the current request prematurely, thus it will never reach a route.

## Implementing your own authentication provider

The actual implementation of an authentication provider is going to vary depending on your requirements but the basic pattern goes as follows

1. A `Before` hook on the [application pipeline](The Application Before, After and OnError pipelines) is used to check for the existence of valid credentials for the incoming request (such as a cookie, headers and so on). If found it will authenticate the user and assign it to the `CurrentUser` property on `NancyContext`
2. A `Before` hook on the [module pipeline](The before and after module hooks) is used to establish that the current request is being performed by an authenticated user. If not then it should deny access and return something like `HttpStatusCode.Unauthorized`
3. An `After` hook on the [application pipeline](The Application Before, After and OnError pipelines) will check for a request that was aborted due to failed authentication, such as keeping an eye out for a response with the `HttpStatusCode.Unauthorized` (401) status code. If detected it will take steps to help the user get authenticated, such as redirecting to a login form or perhaps signal to the client with help of headers

Your mileage may vary but that is the basic flow of an authentication provider.


[<< Part 21. Extending Serialization with Converters](Extending Serialization with Converters) - [Documentation overview](Documentation) - [Part 22. Stateless >>](Stateless authentication)