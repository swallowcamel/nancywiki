Modules are the lynchpin of any given Nancy application. Modules are the one thing you simply cannot avoid because they are where you define the behavior of your application. In fact declaring a single module is the bare minimum requirement for any Nancy application.

A module is created by inheriting from the NancyModule class; it’s as simple as that. Once you have a module you can start defining the behaviors of your application, in the form of routes and the actions they should perform if they are invoked. In short, a module tells Nancy how to behave at runtime.

A module does more than allow you to define application behavior. A module also gives you access to a whole range of useful things such as full information about the current request, access to the context in which the request is being processed, helpers for building up specific kinds of responses (e.g json, xml, files, redirects and so on), rendering views and much more.

You can have as many modules as you like in your application, just as long as there is at least one and that it has at least one route defined.

##Modules are globally discovered

Modules can be declared anywhere you like, just as long as they are available in the application domain at runtime. Nancy will scan and identify all types that are descendants of the NancyModule type. 

This means that you could, for example, define modules in external assemblies. As long as the assembly exists in the application domain at runtime - either through a reference or dynamically loaded - it will be scanned. You could leverage this to reuse behavior across applications, you could even create a nice little nuget for it.

If you think that the scanning sounds expensive, don’t worry. This does not happen on each request to the application, it is part of the startup of any Nancy application and is only performed once and the information is then cached. 

##Using modules to create a root for your routes

One of the small, yet neat, features of a module is the ability to define a module path. You can think of this as a root namespace for all the routes that are defined in the module. Each route will be subordinate to the path of the module. This saves you from having to repeat the common parts of the route patterns and also to nicely group your routes together based on their relationship

```c#
public class ResourceModule : NancyModule
{
    public ResourceModule() : base("/products")
    {
        // would capture routes to /products/list sent as a GET request
        Get["/list"] = parameters => {
            return "The list of products";
        };
    }
}
```

[<< Part 1. Introduction](Introduction) - [Documentation overview](Documentation) - [Part 3. Defining routes >>](Defining routes)