While rendering a view is as simple as invoking the View indexer on the `NancyModule` class, there is quite a lot going on “under the hood” to attempt to locate the correct view template, based on the information provided. This “under the hood” work is actually achieved using a series of “view location conventions”.

The conventions themselves are quite simple. Each convention uses the available “hints”, such as view name, model type and view location context, and produces a series of potential view template names/locations. Nancy will iterate over the conventions, in the order they have been defined, until a view template is located, or until there are no more conventions to use.

## Taking a look at the default conventions

A view location convention is defined by a function of the type `Func<string, dynamic, ViewLocationContext, string>`, and the default conventions that are shipped with Nancy are as follows:

### Root Convention

```c#
(viewName, model, viewLocationContext) => {
    return viewName;
}
```

This convention will basically look for the view in the root of your application. However, if the view name does contain a relative path, the view name will actually point to that path, relative the root of the application, e.g for view `admin/index` it will look for the view in `admin/index`.

### Views Folder Convention
```c#
(viewName, model, viewLocationContext) => {
    return string.Concat("views/", viewName);
}
```
This should be pretty self explanatory; Nancy will look for the view in a views folder located in the root of the application. Again, if you have specified a relative path in your view name, Nancy will look for the view in the path, relative to the views folder, e.g for view `admin/index` it will look for the view in `views/admin/index`. **Note:** do not be alarmed by the use of the hard coded “/” path separator; these are a kind of “virtual” path, rather than physical file system paths, and they always use “/” as a separator.

### Views and Module Path Convention
```c#
(viewName, model, viewLocationContext) => {
    return string.Concat("views/", viewLocationContext.ModulePath, "/", viewName);
}
```
This convention looks a bit more complicated, but it’s really still quite simple. What this is doing is looking for the view, inside a sub-folder of the views folder, where the sub-folder has the same name as the `modulePath` of the module that is being used to execute the current request. So for the view index in a module with a path products it will look for the view in `views/products/index`. Again, if the view name contains a relative path, Nancy will look in the path relative to the `views/modulePath folder`, e.g for view `admin/index`, in a module with a path products, it will look for the view in `views/products/admin/index`.

### Module Path Convention

```c#
(viewName, model, viewLocationContext) => {
    return string.Concat(viewLocationContext.ModulePath, "/", viewName);
}
```
This convention looks for the view inside of a folder with the same name as the modulePath that is located in the root of the application.

### Module Name Convention
```c#
(viewName, model, viewLocationContext) => {
    return string.Concat(viewLocationContext.ModuleName, "/", viewName);
}
```
This convention looks for the view inside of a folder with the name of the module, but with the suffix _Module_ trimmed of, that is located in the root of the application.

### Views and Module Name Convention
```c#
(viewName, model, viewLocationContext) => {
    return string.Concat("views/", viewLocationContext.ModuleName, "/", viewName);
}
```
This convention looks for the view inside of a folder with the name of the module, but with the suffix _Module_ trimmed of, that is located in the views folder of the application.

## Deriving view name from the model type

If you do not provide a view name and only send in the model, then Nancy will derive the view name from the name of the model type. If the name of the model ends with `Model` then it will be stripped off. 

* `Customer` model type name -> `Customer` view name
* `CustomerModel` model type name -> `Customer` view name

## Defining custom conventions

The default conventions definitely won’t suit the need of all applications, or perhaps not always suit your personal preference on where views should be stored. For these scenarios the view conventions, as with everything else in Nancy, are overridable.

To do this, you will need to create a custom bootstrapper and add your conventions (using the function signature described earlier) to the `Conventions.ViewLocationConventions` collection.

For example:

```c#
public class CustomConventionsBootstrapper : DefaultNancyBootstrapper
{
    protected override void ApplicationStartup(TinyIoCContainer container, Nancy.Bootstrapper.IPipelines pipelines)
    {
        this.Conventions.ViewLocationConventions.Add((viewName, model, context) =>
        {
            return string.Concat("custom/", viewName);
        });
    }
}
```
This would add a view location convention which would look for the view inside of a folder called custom that should be located at the root of the application.

The `ViewLocationConventions` collection is a standard list that can be modified, using the normal operators it provides. This enables you to do things such as deleting the default conventions or changing the order in which conventions are invoked.

### Note: 
You can also call `.Insert(0, *convention*)` as well so that it is evaluated first, before all others. Then the existing View Conventions can be used as fall-backs. 

## Defining your own conventions using IConventions

You can also create a class that implements the `IConvention` interface and in the `Initialise` method you add you conventions to the `ViewLocationConventions` property of the conventions that are passed in.

Nancy will locate all implementation of the interface and wire up the conventions, before they are passed onto the `ConfigureConventions` method of the bootstrapper.

[<< Part 8. View engines](View engines) - [Documentation overview](Documentation) - [Part 10. Testing your application >>](Testing your application)