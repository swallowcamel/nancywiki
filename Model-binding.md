Data can be sent to your Nancy application in many ways, for instance as part of the query string, as captured parameters on a route or as the body of a request. Manually working with the data spread out over several places is perfectly fine in some scenarios, but there are times where you want to unify that into something richer - a `model` - which is where Nancy’s “model binding” capabilities come into play.

Nancy, with a single line of code, can collect information from all of the above sources, including richer request bodies such as `JSON` and `XML`, and convert them into an instance of a type of your model.

Although Nancy ships with quite a lot of default binders out of the box, the binding system is easily extensible if you need to support different content types or more complex binding scenarios.
It’s as easy as including a namespace.

Nancy’s model binding is defined as a single extension method on the `NancyModule` type. The extensions are located in the `Nancy.ModelBinding` namespace and adds Bind() and BindTo() methods:

```c#
Foo f = this.Bind();
    
var f = this.Bind<Foo>();

var f = this.BindTo(instance);

```
All three have the exact same functionality, they just provide different ways of doing the same thing. The first two Bind() overloads will create a new instance of the `Foo` type and bind to that, while BindTo() will bind to an existing instance.

## Keeping unwanted information out by blacklisting it

Sometimes you want the model binder to ignore certain information when populating the model from all the various sources (to prevent ‘over posting’ attacks for example). To accommodate this, the model binder can be called with an optional list of blacklisted properties and fields on the model:

```c#
var f = this.Bind<Foo>(f => f.id, f => f.creator, f => f.createddate);
```

The blacklist is a “params” array of Expressions on the model type, where the expressions specify the name of the model properties and fields that should be ignored by the model binder.

or:
```c#
var f = this.Bind<Foo>("id", "creator", "createddate");
```

The blacklist is a “params” array of strings, where the strings represents the name of the model properties and fields that should be ignored by the model binder.

When binding to an typed- array, list or ienumerable. The blacklist is maintained for the elements in the sequence.

## Binding configuration

When using model binding, you can pass in a `BindingConfig` instance to modify the behaviour of the model binder.

Below is a list of available configuration options, provided by the `BindingConfig` type.

Property|Description|Default
--------|-----------|-------
BodyOnly|Whether the binder should be happy once it has bound to the request body. In this case, request and context parameters will not be bound to. If there is no body and this option is enabled, no binding will take place at all.|false
IgnoreErrors|Whether binding error should be ignored and the binder should continue with the next property or field.|false
Overwrite|Whether the binder is allowed to overwrite properties and fields that do not have a default value.|true

There is a short-hand version for declaring that no overwrite should take place: `BindingConfig.NoOverwrite` will return an instance with the property or field value set to `false`.

## Deserializing rich request body payloads

Sometimes you want to send structured data, such as `JSON` and `XML`; in the request body and bind it to your model. The model binder in Nancy supports the notion of body deserializers that provide the functionality required for this.

Out of the box, Nancy ships with two body deserializers, one for JSON and one for XML. The model binder will use the `Content-Type` HTTP header to determine which deserializer should be used for the attached request body.

By default, the standard JSON deserializer will be used to deserialize any "json based" content types such as `application/json`, `text/json` and `application/vnd....+json`. Similarly, the standard XML deserializer will be used for “XML based” content types: `application/xml`, `text/xml` and `application/vnd....+xml`.

As with the other model binders, you can author your own body deserializers and Nancy will automatically detect them and any user defined binders take priority over the built in ones.

**NOTE:** If you encounter the Nancy.Json.JsonSettings.MaxJsonLength Exceeded error because your payload is too high, change that limit in your Bootstrapper in `ApplicationStartup` to be `Nancy.Json.JsonSettings.MaxJsonLength = int.MaxValue;`

## Properties or Fields

Nancy's model binding supports either fields or properties. For the purpose of model binding, these two classes have essentially identical function:

```c#
// Properties
public class Model
{
  public int Value { get; set; }
}
```

```c#
// Fields
public class Model
{
  public int Value;
}
```

There are arguments for either style of class. Using properties means that if you ever need to enforce structure or validation on a property's value down the road, your code is already calling a method, in the form of the ```get``` accessor, and you can alter its implementation as needed. On the other hand, when implicit properties are used (as shown in the example above -- no actual implementation is provided for the ```get``` and ```set``` accessors), the C# compiler picks a name for a hidden backing field for it automatically. This field's name is chosen specifically to avoid collisions with any member names you might type. To do this, it uses characters that are impossible to use in a C# identifier. In particular, it includes angle brackets. If the data type is also used with a ```DataContractSerializer```, this results in a performance hit, because ```DataContractSerializer``` operates specifically on fields, not properties. It finds that backing field and tries to create an XML element with the same name. Since XML elements may not contain ```<``` and ```>``` characters, an exception results, which it handles internally by escaping the field name. (The exception can be seen in debug output when running a program that uses ```DataContractSerializer```, which is the case for most default WCF binding configurations.) The exception and the escaping are unnecessary hits on performance which can be avoided by using field names that are valid XML identifiers as well.

Unless your project has external components referencing it, it is largely unimportant which style you choose, because the syntax in most .NET languages for accessing properties and for accessing fields is identical. Thus, in most circumstances, you can change your model type to use properties or fields and recompile without any other changes required.

## Model binding Checkbox

For automatic model binding checkbox to boolean value, make sure to set the `value="true"` in the checkbox.

```html
<input type="checkbox" name="rememberMe" value="true"/>
```

```c#
public class LoginModel
{
    public bool RememberMe;
}
```

## Model binding against lists
Nancy supports posting in lists of things and binding to them, on the server-side, as before with `this.Bind<T>();`.

Nancy supports binding against lists of items (or arrays or anythings else that implements IEnumerable) in two different ways; as a list on the object your binding against or as a list of objects. 

### Binding against arrays on a single object
If you have a form like this:
```html
<form action="/ArrayOnObject" method="post">
  <input type="text" name="Tags" value="Tag1,Tag2,Tag3"/>
  <input type="text" name="Ints" value="1,2,3,4,4,5,6,3,2,21,1"/>
  <input type="submit" value="Submit"/>
</form>
```
Then you can bind to this class:
```c#
public class Posts
{
  public string[] Tags;
  public int[] Ints;
}
```
with this simple statement:
```c#
var listOfPosts = this.Bind<Posts>();
```
### Binding against lists of objects
Imagine you have a system to enter who commits the most to a OSS-framework. You allow the users of the site to post a bunch of users-names and their number of commits at once. 

Here's an example form with *random* dummy data:
```html
<form action="/SimpleListDemo" method="post">
      User 1:<input type="text" name="Name[0]" value="thecodejunkie" /> 
      Commits <input type="text" name="Commits[0]" value="1068"/>
      <br />
      User 2:<input type="text" name="Name[1]" value="grumpydev" />  
      Commits <input type="text" name="Commits[1]" value="1049"/>
      <br />
      User 3:<input type="text" name="Name[2]" value="jchannon" />  
      Commits <input type="text" name="Commits[2]" value="109"/>
      <br />
      User 4:<input type="text" name="Name[3]" value="prabirshrestha" />  
      Commits <input type="text" name="Commits[3]" value="75"/>
      <br />
      User 5:<input type="text" name="Name[4]" value="phillip-haydon" />  
      Commits <input type="text" name="Commits[4]" value="40"/>
      <br />
      <input type="submit" value="Test the binding thingy"/>
</form>
```
This can then be bound (with ```this.Bind<List<User>>();```) to a list of object of this class:
```c#
public class User
{
   public string Name;
   public int Commits;
}
```

### List delimiters in HTML-forms
Nancy supports two kind of list delimiters for the name of the items in the HTML-form; 
* underscores (`Name_1`, `Name_2` etc.)
* brackets (`Name[1]`, `Name[2]` etc.)


### A Complete Model-binding Example
Here's an end-to-end model binding example that shows the full model + view + module.

    /// <summary>
    /// DTO for creating a new user
    /// </summary>
    public class NewUser
    {
        public string UserName { get; set; }

        public string Email { get; set; }

        public string Password { get; set; }
    }

#### View
```
    ViewBag.Title = "AkkaChat - Register";
}
@inherits Nancy.ViewEngines.Razor.NancyRazorViewBase<AkkaChat.ViewModels.NewUser>

<div class="container-fluid">
    <div class="row-fluid">
        <h1>Register an AkkaChat Account</h1>
        @if (ViewBag.ValidationError != null)
        {
            <div class="alert-error">
                <p>Error! @ViewBag.ValidationError</p>
            </div>
        }

        <form class="form-horizontal" method="POST" action="~/register">
            <div class="control-group">
                <label class="control-label" for="UserName">Username</label>
                <div class="controls">
                    <input type="text" name="UserName"  id="UserName" placeholder="Username">
                </div>
            </div>

            <div class="control-group">
                <label class="control-label" for="Email">Email</label>
                <div class="controls">
                    <input type="text" id="Email" name="Email" placeholder="Email">
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="Password">Password</label>
                <div class="controls">
                    <input type="password" name="Password" id="Password" placeholder="Password">
                </div>
            </div>
            <div class="control-group">
                <div class="controls">
                    <button type="submit" class="btn">Register</button>
                </div>
            </div>
        </form>

    </div>
</div>
```

#### Module
```csharp
/// <summary>
/// Module responsible for handling authentication and account creation
/// </summary>
public class AuthModule : NancyModule
{
    private readonly IMembershipService _membership;

    public AuthModule(IMembershipService membership)
    {
        _membership = membership;

        Get["/register"] = _ =>
        {
            //already logged in
            if (Context.CurrentUser.IsAuthenticated())
                return Response.AsRedirect("~/");
            return View["register"];
        };

        Post["/register", true] = async (x, ct) =>
        {
            NewUser registerAttempt = this.Bind<NewUser>(); //model binding!
            var validationError = "";
            var failedValidation = false;
         
             if (string.IsNullOrEmpty(registerAttempt.UserName))
             {
                 failedValidation = true;
                 validationError += string.Format("Must provide a username!<br>");
             }
             else
             {
                 //check to see if a username is available
                 var userNameAvailable = await _membership.IsUserNameAvailable(registerAttempt.UserName);
                 if (!userNameAvailable)
                 {
                     validationError += string.Format("{0} is already taken. Please pick another username.<br>",
                         registerAttempt.UserName);
                     failedValidation = true;
                 }
             }
             if (string.IsNullOrEmpty(registerAttempt.Password))
             {
                 failedValidation = true;
                 validationError += string.Format("Must provide a password!<br>");
             }
             if (string.IsNullOrEmpty(registerAttempt.Email))
             {
                 failedValidation = true;
                 validationError += string.Format("Must provide an email!<br>");
             }

             if (failedValidation)
             {
                 ViewBag.ValidationError = validationError;
                 return View["register"];
             }

            var registerResult = await _membership.AddUser(
            registerAttempt.UserName, registerAttempt.Email, registerAttempt.Password);

            var registerResult = await _membership.AddUser(
            registerAttempt.UserName, registerAttempt.Email, registerAttempt.Password);

            //success!
            if (!(registerResult is MissingUserIdentity))
            {
                return this.LoginAndRedirect(registerResult.CookieId, DateTime.Now.AddDays(30), "~/");
            }
            else //failure!
            {
                ViewBag.ValidationError = string.Format("Unable to register as {0} - server error.",
                 registerAttempt.UserName);
                return View["register"];
            }

        };

    }
}
```

[<< Part 7. The Application Before, After and OnError pipelines](The Application Before, After and OnError pipelines) - [Documentation overview](Documentation) - [Part 9. Bootstrapper >>](Bootstrapper)