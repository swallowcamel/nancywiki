In Nancy parlance "Static Content" is things like javascript files, css, images etc, but can actually be anything, and Nancy uses a convention based approach for figuring out what static content it is able to serve at runtime. Nancy supports the notion of having multiple conventions for static content and each convention is represented by a delegate with the signature `Func<NancyContext, string, Response>`.

The delegate accepts two parameters; the context of the current request and the requested path, relative to the application root. The output of the delegate is a standard Nancy `Response` object or `null`. A null response means that the convention had no static content to return given the current content and requested path.

Nancy supports multiple static content conventions at once and is shipped with a default convention that will look for files in the `/content` path of your application. The static content handler in Nancy is executed as a `BeforeRequest` filter in the application pipeline.

## Defining your own conventions using the bootstrapper

To define your own static content conventions, using the bootstrapper, you simply inherit a new bootstrapper and override the `ConfigureConventions` method.

The `ConfigureConventions` gives you the opportunity to modify the `StaticContentsConventions` property, which is a list of conventions

```c#
public class CustomBoostrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureConventions(NancyConventions conventions)
    {
        base.ConfigureConventions(conventions);
 
        conventions.StaticContentsConventions.Add((context, path) => {
            // Return your response here or null
        });
    }
}

If that looks a little icky, don't worry, Nancy has a helper that will take care of (hopefully!) the vast majority of any static content convention tweaks.. the StaticContentConventionBuilder:

## Using the StaticContentConventionBuilder
The `StaticContentConventionBuilder` is a helper class that is shipped with Nancy. It produces static content conventions for file system based files (e.g. files on the file system, if you want to embed files in an assembly then see the previous section). It encapsulates a lot of the leg work that you would have to write yourself if you wanted to provide an efficient and secure way of letting clients request files of the file system.

There is only one method, `AddDirectory`, on the class with the following signature

```c#
public static Func<NancyContext, string, Response> AddDirectory(string requestedPath, string contentPath = null, params string[] allowedExtensions)
```

* requestedPath - the path that is actually requested by the client, relative to the application root e.g. /scripts
* contentPath - optional path that contains the content, again, relative the application root. With this parameter it's possible to "map" requests to /scripts to the /javascript folder in your physical directory structure on disk
* allowedExtentions - optional list of extensions that are allowed to be served, so in the above example, you may want to just specify that "js" files are allowed and nothing else.

The resulting convention provides the following features:

* Specify which folder that the static content exists in
* Optionally map the content folder to a _virtual folder_, so that the name of the requested directory is different to that of the actual folder
* Optionally specify which extensions that the convention is valid for
* Caches all file system path evaluations, such as making sure the folder and file actually exists, for improved performance on multiple requests
* Protects against “leaving” the content folder and requesting files that are stored outside the folder and the application itself. Only files in the content folder, or a sub-folder, will be considered valid to return
* Uses the `MimeTypes` list to automatically detect and set the correct content-type of the file response

Using the `StaticContentConventionBuilder`, to create a new convention is really easy

```c#
public class CustomBoostrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureConventions(NancyConventions conventions)
    {
        base.ConfigureConventions(conventions);
  
        conventions.StaticContentsConventions.Add(
            StaticContentConventionBuilder.AddDirectory("assets", @"contentFolder\subFolder")
        );
    }
}
```
## Alternative approaches..

If, for some reason, you don't want to use the methods above, you can also create conventions using the IConventions interface, or bypass the static content conventions completely and just serve your content from a module.

## Defining your own conventions using IConventions

You can also create a class that implements the `IConventions` interface and in the `Initialise` method you add you conventions to the `StaticContentsConventions` property of the conventions that are passed in.

Nancy will locate all implementation of the interface and wire up the conventions, before they are passed onto the `ConfigureConventions` method of the bootstrapper.

## Serving content from a module

You can also use an ordinary `NancyModule` to return static content, by returning responses with the correct body and content-type. Nancy even provide a couple of _response formatters_, to help you out, called `AsJs` and `AsCss`.

The advantage of using a module to serve static content is that you can leverage the full power of a module to implement logic around your static content management, such as making use of module _dependencies_. This can be used when more complex logic is required (such as adding a security layer on top).

However, there are disadvantages to using modules for static content management. Because it is a module, the request, for the static content, is subjected to the same request life-cycle as any other request.

This means that the request needs to go through things such as route resolving which can have a performance impact on your application.

[<< Part 11. The root path](The root path) - [Documentation overview](Documentation) - [Part 13. Diagnostics](Diagnostics) >>