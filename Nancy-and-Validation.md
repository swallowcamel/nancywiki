Nancy and FluentValidation
==========================
You can install the FluentValidation from the [NuGet packages](https://www.nuget.org/packages/Nancy.Validation.FluentValidation/):

    Install-Package Nancy.Validation.FluentValidation

The version of FluentValidation that is required by this library however is rather old, so you'll need to update it as the public key appears to have expired:

    Update-Package FluentValidation

This REST API module allows us to add a new user:

```c#
public class AddUserRequest
{
    public string Username { get; set; }
    public string Password { get; set; }
}

public interface IUserService
{
    void AddUser(string username, string password);
}

public class UserModule : NancyModule
{
    private readonly IUserService _userService;

    public UserModule(IUserService userService)
    {
        _userService = userService;
        Put["/users"] = AddUser;
    }

    private object AddUser(dynamic parameters)
    {
        var userInfo = this.Bind<AddUserRequest>();
        _userService.AddUser(userInfo.Username, userInfo.Password);

        return new
        {
            Result = "User Added"
        };
    }
}
```

However, if we specify a blank username or password, it will add the incorrect data to the database, probably causing an error further down the line. We can add validation to ensure this does not happen.

Firstly, define a validation class:

```c#
public class AddUserValidator : AbstractValidator<AddUserRequest>
{
    public AddUserValidator()
    {
        RuleFor(request => request.Username).NotEmpty().WithMessage("You must specify a username.");
        RuleFor(request => request.Password).NotEmpty().WithMessage("You must specify a password.");
        RuleFor(request => request.Password).Length(8, 20).WithMessage("The password must be between 8 and 20 characters long.");
    }
}
```

There is no need to register it anywhere as it is automatically detected.

We can now check that the input model is valid by modifying the AddUser method:

```c#
private object AddUser(dynamic parameters)
{
    var userInfo = this.Bind<AddUserRequest>();
    var validationResult = this.Validate(userInfo);

    if (!validationResult.IsValid)
    {
        return Negotiate.WithModel(validation).WithStatusCode(HttpStatusCode.BadRequest);
    }

    _userService.AddUser(userInfo.Username, userInfo.Password);

    return new
    {
        Result = "User Added"
    };
}
```
This will return the error status code and it will return a diagnostic error message to the client that called the service.

For more information on validation rules, you can read about the [FluentValidation library](https://github.com/JeremySkinner/FluentValidation).

Assembly Scanning
-----------------
If your validation types are not located in your main web app assembly, you will see that your input data is not validated. You have a couple of options in this case. The easiest way to do this is to. Firstly, you can add this to your Properties.cs file in your validation assembly:

```c#
[assembly: IncludeInNancyAssemblyScanning]
```

If you don't want your validation assembly to have a reference to Nancy, you can configure the AppDomainAssemblyTypeScanner in your Bootstrap class to consider it:

```c#
public class Bootstrap : DefaultNancyBootstrapper
{
    public Bootstrap()
    {
        AppDomainAssemblyTypeScanner.LoadAssemblies("MyApplication.dll");
    }
}
```