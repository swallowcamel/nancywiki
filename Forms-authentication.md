This document provides an overview on how to enable forms authentication in your Nancy application. For a full working sample, please refer to the `Nancy.Demo.Authentication.Forms` project in the Nancy solution.

To enable forms authentication, in your application, you need to complete the following steps

1. Install the Nancy.Authentication.Forms package
1. Implement an `IUserMapper`
1. Implement routes to handle login and logout
1. Configure and enable Forms Authentication

## The user mapper

The user mapper is responsible for mapping a user from an identifier. The identifier is the token that will be put in the authentication cookie which will be used to re-establish the identity of the user that is performing the request, so that you do not need to enter your credentials for each request.

We have chosen to use a GUID as the identifier. The reason for this is that using something like the username of id is a potential vulnerability, because, if the cookie encryption was somehow compromised,  it would be easy to spoof the identity of another user by guessing their username or id.

The identifier should never be read by a human, so making it a GUID also discourages that it is ever displayed anywhere, and each GUID is both unique and non-sequential, making it a lot harder to spoof the identity of someone else.

It is also important to know that the identifier should be treated permanent for the user that it was generated for and will be reused across requests and application sessions. 

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

A single method whose sole responsibility is to take the GUID identifier and return the corresponding user. If the identifier could not be mapped to a user then `null` should be returned. For more information about the `IUserIdentity` interface, refer to the [Authentication overview].

## Modifying your application to handle forms authentication

Once you have implemented an `IUserMapper` then it is time to add the resources, to your application, that are responsible for handling the login and logout procedures. These are the implemented using normal routes, and should be placed in a module that does not require the user to be authenticated to access.

Below is an example on the basic structure of such a module. Please note that the paths of the resources, and the name of the module, can be anything; it’s not necessary to follow the naming in this example:

```c#
public class LoginModule : NancyModule
{
    public LoginModule()
    {
        Get[“/login”] = parameters => {
            // Called when the user visits the login page or is redirected here because
            // and attempt was made to access a restricted resource. It should return
            // the view that contains the login form
        };

        Get[“/logout”] = parameters => {
            // Called when the user clicks the sign out button in the application. Should
            // perform one of the Logout actions (see below)
        };

        Post[“/login”] = parameters => {
            // Called when the user submits the contents of the login form. Should
            // validate the user based on the posted form data, and perform one of the
            // Login actions (see below)
        };
    }
}
```

When the user either login or logout, certain actions needs to be taken, such as creating or removing the authentication cookie and to redirect the user to the appropriate location. To aid with these common tasks, there are several extension methods (located in the `Nancy.Authentication.Forms` namespace) that you can use

- `LoginAndRedirect` - Logs the user in an redirects the user back to the url that they came from. You can specify a fallback url that will be used if no redirect url was available
- `LoginWithoutRedirect` - Logs the user in and returns a response with status code 200 (OK)
- `Login` will call the `IsAjaxRequest` extension method on the current request and perform a `LoginAndRedirect` if it was not an Ajax call, otherwise it will perform `LoginWithoutRedirect` 
- `LogoutAndRedirect` - Logs the user out and redirects to the provided url
- `LogoutWithoutRedirect` - Logs the user our and returns a response with status code 200 (OK)
- `Logout` will call the `IsAjaxRequest` extension method on the current request and perform a `LogoutAndRedirect` if it was not an Ajax call, otherwise it will perform `LogoutWithoutRedirect`

Note1: The `IsAjaxRequest` extension method (found in Nancy.Extensions.RequestExtensions) will look for the `X-Requested-With` header and return true if it contains the value `XMLHttpRequest`

Note 2: Please make sure the routes you define for displaying the login page and taking login credentials are not secured behind a RequiresAuthentication call - you may laugh, but this is quite a common mistake :-)

## Enabling forms authentication

The only thing left to do now is actually enable Forms Authentication, this is done by simply adding a line like this to your bootstrapper:

```c#
FormsAuthentication.Enable(pipelines, formsAuthConfiguration);
```

It can either be added to the `ApplicationStartup` or `RequestStartup`. Which you should choose most often depends on your implementation of the `IUserMapper`, that is, should be user mapper have an application or request lifetime.

The `formsAuthConfiguration` variable, that is passed into `FormsAuthentication.Enable` method, is an instance of the `FormsAuthenticationConfiguration` type, which enables you to customize the behavior of the forms authentication provider.

For instance, here is a basic configuration

```c#
var formsAuthConfiguration =
    new FormsAuthenticationConfiguration()
    {
        RedirectUrl = "~/login",
        UserMapper = container.Resolve<IUserMapper>(),
    };
```

It configures the Forms Authentication to redirect the user to `~/login` when an unauthorised user attempts to gain access to a protected resource and it tells it which user mapper it should use. 

Depending on if this is placed in `ApplicationStartup` or `RequestStartup` the mapper will have different lifetimes. Also, please note that the method for resolving the mapper, from the container, will vary depending on which bootstrapper you are using.

The following configurations can be made

- `RedirectingQuerystringKey` - The name of the querystring parameter that will contain the URL of the secure resource that was being accessed while being unauthorized. This is used to redirect the user back once the login was successful. The default name is `returnUrl`
- `RedirectingUrl` - The, relative, url that user should be redirected to when trying to access a secure resource when not being authenticated. This is usually your login page and should be in the format `~/login`
- `UserMapper` - The `IUserMapper` that should be used during authentication
- `RequiresSSL` - Determines if the forms authentication cookie can only be accessed over a secure request or not. The default is false
- `DisableRedirect` - Determines whether to redirect to the login page during an unauthorized access
- `CryptographyConfiguration` - The `CryptographyConfiguration.Default` that should be used when working with the forms authentication cookie. If this is not specified then `CryptographyConfiguration.Default` will be used as the default.

## A word on cryptography

If you do not specify a value for the `CryptographyConfiguration` property on the `FormsAuthenticationConfiguration` object, then it will default to using `CryptographyConfiguration.Default`.

The default behavior uses the `RandomKeyGenerator` which means that a new crypto key is generated each time the application starts, so if your application is restarted, or the appdomain recycled, then user’s authentication cookies will become invalid; this will also be the case between machines in a load balanced environment. Don’t panic though! Changing this configuration is simple to do though, for more in depth information read about the  [Cryptographic configurations].

For this reason you should always use your own `CryptographyConfiguration` and below is an example of what that could look like, when used together with forms authentication.

```c#
var cryptographyConfiguration = new CryptographyConfiguration(
    new RijndaelEncryptionProvider(new PassphraseKeyGenerator("SuperSecretPass", new byte[] { 1, 2, 3, 4, 5, 6, 7, 8 })),
    new DefaultHmacProvider(new PassphraseKeyGenerator("UberSuperSecure", new byte[] { 1, 2, 3, 4, 5, 6, 7, 8 })));

this.config = 
    new FormsAuthenticationConfiguration()
    {
        CryptographyConfiguration = cryptographyConfiguration,
        RedirectUrl = "/login",
        UserMapper = container.Resolve<IUserMapper>(),
    };
```