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

Sometimes you want the model binder to ignore certain information when populating the model from all the various sources (to prevent ‘over posting’ attacks for example). To accommodate this, the model binder can be called with an optional list of blacklisted properties on the model:

```c#
var f = this.Bind<Foo>("id", "creator", "createddate");
```

The blacklist is a “params” array of strings, where the strings represents the name of the model properties that should be ignored by the model binder.

## Deserializing rich request body payloads

Sometimes you want to send structured data, such as `JSON` and `XML`; in the request body and bind it to your model. The model binder in Nancy supports the notion of body deserializers that provide the functionality required for this.

Out of the box, Nancy ships with two body deserializers, one for JSON and one for XML. The model binder will use the `Content-Type` HTTP header to determine which deserializer should be used for the attached request body.

By default, the standard JSON deserializer will be used to deserialize any "json based" content types such as `application/json`, `text/json` and `application/vnd....+json`. Similarly, the standard XML deserializer will be used for “XML based” content types: `application/xml`, `text/xml` and `application/vnd....+xml`.

As with the other model binders, you can author your own body deserializers and Nancy will automatically detect them and any user defined binders take priority over the built in ones.

## Model binding Checkbox

For auto model binding checkbox to boolean value, make sure to set the `value="true"` in the checkbox.

```html
<input type="checkbox" name="rememberMe" value="true"/>
```

```c#
public class LoginModel
{
    public bool RememberMe { get; set; }
}
```

[<< Part 5. The before and after module hooks](The before and after module hooks) - [Documentation overview](Documentation) - [Part 7. Bootstrapper >>](Bootstrapper)