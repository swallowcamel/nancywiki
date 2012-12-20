The Super Simple View Engine, also known as SSVE, is a regex (the implementation uses regular expressions to perform substitutions) based view engine that was designed to support simple templating scenarios, so a lot of the features you see in other engines may not be available.

There is no need to install a separate Nuget to use the engine as it is embedded into the main Nancy assembly, and will automatically be wired up and ready to use in your application. The engine handles views with either `sshtml`, `html` or `htm` file extensions.

Models can either be standard types, or `ExpandoObjects` (or, in reality, any other object implementing `IDynamicMetaObjectProvider` that implements `IDictionary<string, object>` to access its properties).

All commands have an optional semi-colon delimiter which can be used to remove ambiguity. Any `[.Parameters]` parameter can be multiple levels deep (e.g. `This.Property.That.Property`).

As SSVE is a regular expression based view engine there is no “code execution”, so you cannot specify an arbitrary chunk of your own code to execute. The built in syntax/commands that you can use are as follows.

### Standard variable substitution
Replaces with the string representation of the parameter, or the model itself if a parameter is not specified. If the substitution can not be performed, for instance if you specify an invalid model property, it will be substituted with `[Err!]`

Syntax
```
@Model[.Parameters]
```

Example
```
Hello @Model.Name, your age is @Model.User.Age
```

### Iterators
Enables you to iterate over models that are collection. Iterators cannot be nested 

Syntax
```
@Each[.Parameters]
   [@Current[.Parameters]]
@EndEach
```

`@Each` will implicitly be associated with the model and for each iteration the `@Current` will represent the current item in the collection. `@Current` can be used multiple times in the iterator block, and is accessed in the same way as `@Model`.

Example
```
@Each.Users
   Hello @Current.Name!
@EndEach
```

### Conditionals
Parameters must be a boolean (see Implicit Conditionals below). Nesting of @If and @IfNot statements is not supported.


Syntax: 
```
@If[Not].Parameters
   [contents]
@EndIf
```

Example
```
@IfNot.HasUsers
   No users found!
@EndIf
```

### Implicit Conditionals
If the model has property that implements `ICollection` then you can use an implicit conditional. The implicit conditional syntax is the same as a normal conditional, but the `Parameters` part can have a `Has`-prefix. The conditional will be true if the collection contains items, and false if it does not or if it is null.

Syntax
```
Has[CollectionPropertyName]
```

Example
```
@If.HasUsers
   Users found!
@EndIf
```

The above example will expand to "Users found!" if the model has a collection called `Users` and it contains items; if the collection is empty then the text would not be displayed.

### HTML Encoding
Both the `@Model` and `@Current keywords` (with or without parameters) can have an optional `!` operator, after the `@`, to HTML encode the output.

Syntax
```
@!Model[.Parameter]
@!Current[.Parameter]
```

Example
```
@!Model.Test

@ForEach
   @!Current.Test
@EndEach
```

### Partials
Renders a partial view. A property of the current model can be specified to be used as the partial view's model, or it may be omitted to use the current view's model instead. The file extension of the view is optional.

Syntax
```
@Partial['<view name>'[, Model.Property]]
```

Example
```
// Renders the partial view with the same model as the parent
@Partial['subview.sshtml'];

// Renders the partial view using the User as the model
@Partial['subview.sshtml', Model.User];
```
### Master pages and sections
You can put shared layout in a master page and declare content sections that will be populated by the views. It is possible to have nested master pages and you are not obligated to provide content for all of the content sections.

The master pages will have access to the `@Model` of the view and the file extension is optional when specifying the name of the master to use in your view.

You can use the `@Section` tag multiple times and is used to both declare a content section, in a master page, and to define the content blocks of a view.

Syntax
```
@Master['<name>']

@Section['<name>']
[@EndSection]
```

Example
```
// master.sshtml
<html>
<body>
@Section['Content'];
</body>
</html>

// index.sshtml
@Master['master.sshtml']

@Section['Content']
   This is content on the index page
@EndSection
```

### Anti-forgery token
Renders our an anti-forgery token, on the page, in an hidden input to prevent cross-site request forgery attacks. The token will automatically be validated when a new request is posted to the server (assuming CSRF protection hasn’t been turned off).

Syntax
```
@AntiForgeryToken
```

Example
```
@AntiForgeryToken
```

### Path expansion
Expands a relative paths to a fully qualified URL. 

Syntax
```
@Path['<relative-path>']
```

Example
```
@Path['~/relative/url/image.png']
```