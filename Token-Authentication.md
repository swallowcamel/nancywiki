This document provides an overview on how to enable token authentication in your Nancy application. For a full working sample, please refer to the `Nancy.Demo.Authentication.Token` project in the Nancy solution.

# Meet Nancy Token Authentication

The Nancy.Authentication.Token project was built for use by heterogeneous clients (iOS apps, Android apps, Angular SPA apps, etc.) that all communicate with the same back-end Nancy application.

## Rationale

Token authentication and authorization was built with the following requirements:

* No Cookies (since not all client apps are web browsers)
* Avoid retrieving users and permissions from a backend data store once the user has been authenticated/authorized
* Allow client apps to store a token containing the current user's credentials for resubmission on subsequent requests (after first authenticating)
* Prevent rogue clients from simply generating their own spoofed credentials by incorporating a one-way hashing algorithm that ensures the token has not been tampered with
* Use server side keys for token hash generation with a configurable key expiration interval
* Use file system storage of server-side token generation private keys to allow keys to survive an application restart or an app pool recycle. Note: an "in memory" option is available primarily for testing, but could be used in a situation where expiring all user sessions on an application restart is acceptable behavior.

## Usage

### Nancy Configuration

Token Authentication can be wired up in a simliar fashion to other available forms of Nancy authentication.

```csharp
    public class Bootstrapper : DefaultNancyBootstrapper
    {
        protected override void RequestStartup(TinyIoCContainer container, IPipelines pipelines, NancyContext context)
        {
            TokenAuthentication.Enable(pipelines, new TokenAuthenticationConfiguration(container.Resolve<ITokenizer>()));
        }
    }
```

You will need to provide your own form of initial user authentication. This can use your own custom implementation that queries
from a database, from an AD store, from a webservice, or any other form you choose. It could also use another form of Nancy authentication (Basic with an IUserValidator implementation
for example).

Tokens are generated from an `IUserIdentity` and a `NancyContext` by an implementation of `ITokenizer`. The 
default implementation is named `Tokenizer` and provides some configuration options. By default, it generates a token
that includes the following components:

* User name
* Pipe separated list of user claims
* UTC now in ticks
* The client's "User-Agent" http header value (required)

It is recommended that you configure the Tokenizer to use an additional piece of information that can uniquely identify 
the client device. 

The following code shows an example of how you can perform the initial user authorization and return the generated token to the client.

```csharp
    public class AuthModule : NancyModule
    {
        public AuthModule(ITokenizer tokenizer)
            : base("/auth")
        {
            Post["/"] = x =>
                {
                    var userName = (string)this.Request.Form.UserName;
                    var password = (string)this.Request.Form.Password;

                    var userIdentity = UserDatabase.ValidateUser(userName, password);

                    if (userIdentity == null)
                    {
                        return HttpStatusCode.Unauthorized;
                    }

                    var token = tokenizer.Tokenize(userIdentity, Context);

                    return new
                        {
                            Token = token,
                        };
                };

            Get["/validation"] = _ =>
                {
                    this.RequiresAuthentication();
                    return "Yay! You are authenticated!";
                };

            Get["/admin"] = _ =>
            {
                this.RequiresClaims(new[] { "admin" });
                return "Yay! You are authorized!";
            };
        }
    }
```
### Client Configuration

Once your client has received a token, it must send the token as an HTTP header with each subsequent request.

```
Authorization: Token [your token goes here] (without square brackets)
``` 

## Under the Covers

If you're seeking a deeper understanding of how Nancy.Authentication.Token works and the theory behind it, then this section is for you.

The first thing to understand is that generated tokens are not stored on the server. All that is stored on the server are keys used for generating the tokens. Storing the actual tokens on the server would defeat one of the purposes of this library which is to avoid any fetching of stored credentials on the server once a user is authenticated. All that is stored in "keyChain.bin" is the key that is used to generate user tokens. At any given time, there are at most two keys in the chain - the most recent key used to generate tokens, and the one generated immediately before it. This is to allow for the 24 hour time window of token expiration to work properly since a user could have their token generated at any time during the lifecycle of the key. If the key is already 23 hours old, you don't want the user to have to log in again in an hour. Once a key is 24 hours old, it sticks around to authenticate tokens that it created until they are 24 hours old, but is no longer used to generate any new tokens. At 48 hours (24 * 2) the older key is removed from the chain. So there are at most two keys: the current key for generating new tokens and authenticating tokens it has generated, and the "previous" key which is not used to generate new tokens, but is used to authenticate "older" tokens that it generated until the tokens themselves expire at 24 hours. In summary, the keyChain.bin doesn't store tokens, just the keys used for generating tokens.

In a critical situation, you can expire the keys which will cause all tokens generated with that key to immediately expire. But the Token Auth implementation is meant to help avoid paying the relatively high cost of authorization and authentication against a backend store on each request. It was originally created to work around a requirement of authenticating users against Active Directory which had a high cost (at least high by our standards and in our context). By caching the users credentials in a token, there is no further need to do an AD authenticate/authorize on each request. Of course, the same principle applies if you have to query a database or other resource for user authentication/authorization. The cost may not be huge but simply validating a token will always incur less of a cost.

If users of this project are interested in an optional server token store, or username "blacklist" for expiring users or forcing a re-login, then please leave a comment on the original commit for this project: [https://github.com/NancyFx/Nancy/commit/9ae0a5494bc335c3d940d730ae5d5f18c1018836](https://github.com/NancyFx/Nancy/commit/9ae0a5494bc335c3d940d730ae5d5f18c1018836)

## Contributors

Nancy.Authentication.Token was originally created by the crack development team at [Lotpath](http://lotpath.com) ([Lotpath on github](http://github.com/Lotpath)).

[<< Part 21. Token](Token Authentication) - [Documentation overview](Documentation)