**TL;DR: stick stuff in `/Content` .. done.** 

In Nancy parlance "Static Content" is things like javascript files, css, images etc, but can actually be anything, and Nancy uses a convention based approach for figuring out what static content it is able to serve at runtime. Nancy supports the notion of having multiple conventions for static content and each convention is represented by a delegate with the signature `Func<NancyContext, string, Response>`.

The delegate accepts two parameters; the context of the current request and the requested path, relative to the application root. The output of the delegate is a standard Nancy `Response` object or `null`. A null response means that the convention had no static content to return given the current content and requested path.

Nancy supports multiple static content conventions at once and is shipped with a default convention that will look for files in the `/Content` path of your application. The static content handler in Nancy is executed as a `BeforeRequest` filter in the application pipeline.

## Defining your own conventions using the bootstrapper

To define your own static content conventions, using the bootstrapper, you simply inherit a new bootstrapper and override the `ConfigureConventions` method.

The `ConfigureConventions` gives you the opportunity to modify the `StaticContentsConventions` property, which is a list of conventions

```c#
public class CustomBootstrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureConventions(NancyConventions conventions)
    {
        base.ConfigureConventions(conventions);
 
        conventions.StaticContentsConventions.Add((context, path) => {
            // Return your response here or null
        });
    }
}
```
If that looks a little icky, don't worry, Nancy has a helper that will take care of (hopefully!) the vast majority of any static content convention tweaks.. the StaticContentConventionBuilder:

## Using the StaticContentConventionBuilder
The `StaticContentConventionBuilder` is a helper class that is shipped with Nancy. It produces static content conventions for file system based files (e.g. files on the file system, if you want to embed files in an assembly then see the previous section). It encapsulates a lot of the leg work that you would have to write yourself if you wanted to provide an efficient and secure way of letting clients request files of the file system.

There are two methods that produce static content conventions
* `AddDirectory` - For mapping directories
* `AddFile` - For mapping individual files

### Mapping a directory using AddDirectory

When using `AddDirectory` you are creating a mapping between a virtual and a physical folder on the filesystem. For instance you may store your static content in _ /assets_ on the file system, but wish for all requests, for that content, to be sent to _/public_

Forward slashes should _always_ be used as the path separator, no matter on which platform the application is run. Nancy will ensure that, during run-time, these paths will be converted to valid paths for the platform that the application is running on.

It is important to be aware that you are mapping to a static content root folder and that it is valid to request resources from any sub-folder in that folder. For example if you map _/public_ to _/assets_, you can then add a _javascript_ sub-folder, in _/assets_ which can then be requested using _/public/javascript/_ 

It is not permitted to map to the root of your application. Any attempt at doing so will result in an _ArgumentException_ being thrown. The reason for this is that it would make it possible to request **any** file in your application if you are not careful.

The `AddDirectory` method has the following signature

```c#
public static Func<NancyContext, string, Response> AddDirectory(string requestedPath, 
                                                                string contentPath = null, 
                                                                params string[] allowedExtensions)
```

* `requestedPath` - the path that is actually requested by the client, relative to the application root e.g. /scripts. 
* `contentPath` - optional path that contains the content, again, relative the application root. With this parameter it's possible to "map" requests to /scripts to the /javascript folder in your physical directory structure on disk.
* `allowedExtentions` - optional list of extensions that are allowed to be served, so in the above example, you may want to just specify that "js" files are allowed and nothing else.

The resulting convention provides the following features:

* Specify which folder that the static content exists in
* Optionally map the content folder to a _virtual folder_, so that the name of the requested directory is different to that of the actual folder
* Optionally specify which extensions that the convention is valid for
* Caches all file system path evaluations, such as making sure the folder and file actually exists, for improved performance on multiple requests
* Protects against “leaving” the content folder and requesting files that are stored outside the folder and the application itself. Only files in the content folder, or a sub-folder, will be considered valid to return
* Uses the `MimeTypes` list to automatically detect and set the correct content-type of the file response

### Mapping individual files with AddFile

The `AddFile` method enables you to map individual file names and locations. For example if you do not wish to store your _robots.txt_ file in the root of your application, but instead store it in your _/asserts_ folder, you could map the location of the file and Nancy would forward incoming requests, for that file, to the right location.

It is also possible to map extensionless files to physical files, for example _ /style_ could be mapped to _/assets/style.css_. If a file request is then sent to _/style_, then the _/assets/style.css_ file will be returned in its place.

The `AddFile` method has the following signature

```c#
public static Func<NancyContext, string, Response> AddFile(string requestedFile, string contentFile)
```

* `requestedFile` - The name, including path, of the file that is requested by the client
* `contentFile` - The name, including path, of the file that will be returned in place of the requested file

The resulting convention provides the following features:

* Caches all file system path evaluations, such as making sure the folder and file actually exists, for improved performance on multiple requests
* Protects against “leaving” the content folder and requesting files that are stored outside the folder and the application itself
* Uses the `MimeTypes` list to automatically detect and set the correct content-type of the file response

### Defining the new conventions for your application

Using the `StaticContentConventionBuilder`, to create new conventions is really easy

```c#
public class CustomBoostrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureConventions(NancyConventions conventions)
    {
        base.ConfigureConventions(conventions);
  
        conventions.StaticContentsConventions.Add(
            StaticContentConventionBuilder.AddDirectory("assets", @"contentFolder/subFolder")
        );
    }
}
```
## Alternative approaches..

If, for some reason, you don't want to use the methods above, you can also create conventions using the IConventions interface, or bypass the static content conventions completely and just serve your content from a module.

## Defining your own conventions using IConventions

You can also create a class that implements the `IConventions` interface and in the `Initialise` method you add your conventions to the `StaticContentsConventions` property of the conventions that are passed in.

Nancy will locate all implementations of the interface and wire up the conventions, before they are passed onto the `ConfigureConventions` method of the bootstrapper.

## Serving content from a module

You can also use an ordinary `NancyModule` to return static content, by returning responses with the correct body and content-type. Nancy even provides a few _response formatters_, to help you out, called `AsFile`, `AsImage`, `AsJson`, `AsText`, `AsXml` and `AsRedirect`.

The advantage of using a module to serve static content is that you can leverage the full power of a module to implement logic around your static content management, such as making use of module _dependencies_. This can be used when more complex logic is required (such as adding a security layer on top).

However, there are disadvantages to using modules for static content management. Because it is a module, the request, for the static content, is subjected to the same request life-cycle as any other request.

This means that the request needs to go through things such as route resolving which can have a performance impact on your application.

## Letting IIS handle static content

Sometimes you may want to allow IIS to handle static content for you rather than letting Nancy do it, maybe for performance reasons or possibly something gets miss-configured and it's not working. You can tell IIS to not use Nancy for static content by removing the handler for a specific path:

```xml
<location path="images">
  <system.webServer>
    <handlers>
      <remove name="Nancy"/>
    </handlers>
  </system.webServer>
</location>
```

You can change the path to be 'images', 'scripts', 'content, or what ever you need, just be sure you don't have any routes defined which need to use this path, they will no longer work since you're basically telling IIS to not pass any of those requests to the Nancy Handler.

## Extra steps required when using Microsoft.Owin.Host.SystemWeb

When you are hosting Nancy using `Microsoft.Owin.Host.SystemWeb` you need some additional configuration to get it working in IIS.

First of all, in your OWIN Startup.cs you will need to add `app.UseStageMarker(PipelineStage.MapHandler)`, for example:

```c#
using Microsoft.Owin.Extensions;
using Owin;

public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseNancy();
        app.UseStageMarker(PipelineStage.MapHandler);
    }
}
```

You will also have to add the following to your `Web.config`:

```
<system.webServer>
    <modules runAllManagedModulesForAllRequests="true" />
</system.webServer>
```

Without this additional configuration, the `StaticFileModule` from OWIN would intercept the request and return a 404.

[<< Part 14. The root path](The root path) - [Documentation overview](Documentation) - [Part 16. Adding a custom FavIcon >>](Adding a custom FavIcon)