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
var f = this.Bind<Foo>(f => f.id, f => f.creator, f => f.createddate);
```

The blacklist is a “params” array of Expressions on the model type, where the expressions specify the name of the model properties that should be ignored by the model binder.

or:
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

## Model binding against lists
Nancy supports posting in lists of things and binding to them, on the server-side, as before with ```c#this.Bind<T>();```.

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
public class Example
{
  public string Title { get; set; }
  public DateTime Published { get; set; }
  public string[] Tags { get; set; }
  public int[] Ints { get; set; }
}
```
### Binding against lists of objects
Imagine you have a system to enter who commits the most to a OSS-framework. You allow the users of the site to post a bunch of users-names and their number of commits at once. 

Here's an example form with random dummy data:
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

This can then be bound (with ```c#this.Bind<T>();```) to a list of object of this class:
```c#
public class User
{
   public string Name { get; set; }
   public int Commits { get; set; }
}
```

### List delimiters in HTML-forms
Nancy supports two kind of list delimiters for the name of the items in the HTML-form; 
* underscore (Name_1, Name_2 etc.)
* brackets (Name[1], Name[2] etc.)

You can start the numbering on whatever number you fancy and you don't have to keep the series in order. Nancy got you covered. 


[<< Part 5. The before and after module hooks](The before and after module hooks) - [Documentation overview](Documentation) - [Part 7. Bootstrapper >>](Bootstrapper)