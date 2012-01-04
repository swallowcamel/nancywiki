Nancy uses a convention based approach for figuring out what static content it is able to server at runtime. Nancy supports the notion of having multiple conventions for static content and each convention is represented by a delegate with the signature `Func<NancyContext, string, Response>`.

The delegate accepts two parameters; the context of the current request and the requested path, relative to the application root. The output of the delegate is a standard Nancy `Response` object or `null`. A null response means that the convention had no static content to return given the current content and requested path.

Nancy supports multiple static content conventions at once and is shipped with a default convention that will look for files in the `/content` path of your application. The static content handler in Nancy is executed as a `BeforeRequest` filter in the application pipeline.

## Defining your own conventions using the boostrapper

To define your own static content conventions, using the bootstrapper, you simply inherit a new bootstrapper and override the `ConfigureConventions` method.

The `ConfigureConventions` gives you the opportunity to modify the `StaticContentsConventions` property, which is a list of conventions

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

## Defining your own conventions using IConventions

You can also create a class that implements the `IConventions` interface and in the `Initialise` method you add you conventions to the `StaticContentsConventions’ property of the conventions that are passed in.

Nancy will locate all implementation of the interface and wire up the conventions, before they are passed onto the `ConfigureConventions` method of the bootstrapper.

## Using the StaticContentConventionBuilder
The `StaticContentConventionBuilder` is a helper class that is shipped with Nancy. It produces static content conventions for file system based files. It encapsulates a lot of the leg work that you would have to write yourself if you wanted to provide an efficient and secure way of letting clients request files of the file system.

There is only one method, `AddDirectory`, on the class with the following signature

`public static Func<NancyContext, string, Response> AddDirectory(string requestedPath, string contentPath = null, params string[] allowedExtensions)`

The following are provided by the conventions that the builder produces

* Specify which folder that the static content exists in
* Optionally map the content folder to a _virtual folder_, so that the name of the requested directory is different to that of the actual folder
* Optionally specify which extensions that the convention is valid for
* Caches all file system path evaluations, such as making sure the folder and file actually exists, for improved performance on multiple requests
* Protects against “leaving” the content folder and requesting files that are stored outside the folder and the application itself. Only files in the content folder, or a sub-folder, will be considered valid to return
* Uses the `MimeTypes` list to automatically detect and set the correct content-type of the file response

## Serving content from a module

You can also use an ordinary `NancyModule` to return static content, by returning responses with the correct body and content-type. Nancy even provide a couple of _response formatters_, to help you out, called `AsJs` and `AsCss`.

The advantage of using a module to server static content is that you can leverage the full power of a module to implement logic around your static content management, such as making use of module _dependencies_. This can be used when more complex logic is required (such as adding a security layer on top).

However, there are disadvantages to using modules for static content management. Because it is a module, the request, for the static content, is subjected to the same request life-cycle as any other request.

This means that the request needs to go through things such as route resolving which can have a performance impact on your application.

[<< Part 11. The root path](The root path) - [Documentation overview](Documentation)