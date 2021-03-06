Right smack in the center of Nancy lives this mythical unicorn that we like to call the `Bootstrapper`. Rather than being one monolithic framework, Nancy is actually lots of little pieces of functionality that are composed together when the application starts up. Configuration of this composition, including swapping out any of the default “pieces of the puzzle”, is done via the bootstrapper.

The bootstrapper is also responsible for some of the magic that you are treated to when working with the framework, such as automatic discovery of modules, custom model binders, dependencies and much more.

In essence the bootstrapper is a friendly syntax, or [Domain Specific Language (DSL)](http://en.wikipedia.org/wiki/Domain-specific_language), on top of an [Inversion of Control Container](http://en.wikipedia.org/wiki/Inversion_of_Control) that helps out with the component discovery and composition of the framework at runtime. Don’t worry if any of these terms sound new or scary, the super-duper-happy-path ensures that everything “just works” out of the box, without having to worry about the underlying container, and still allows very granular configuration/customisation as and when you require it.

The default implementation that is shipped with Nancy sits on top of the [TinyIoC](https://github.com/grumpydev/TinyIoC) IoC container; however, just as the bootstrapper lets you swap out pretty much any component in Nancy, the bootstrapper itself can be replaced with an implementation using a container of your choice. We ship implementations for all the major containers, available via [Nuget](https://nuget.org/packages?q=Nancy.Bootstrappers) or [our TeamCity server](http://nancy-ci.cloudapp.net/) (for latest builds), but if there’s a container that’s not currently shipped that you think we should support - feel free to send a pull request :smile:

## Basic bootstrapper modifications

When you want to change the runtime behavior of Nancy you are going to be doing this through a custom bootstrapper. Fortunately this doesn’t mean you have to implement a bootstrapper from scratch each time, but instead you derive from the “base” bootstrapper you are using and override one of the methods / properties. In this example the ApplicationStartup method is overridden:

```c#
public class CustomBootstrapper : DefaultNancyBootstrapper
{
    protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines)
    {
         // your customization goes here
    }
}
```

## Finding the right bootstrapper

When the application starts up, Nancy looks for a custom bootstrapper. By default, it scans all assemblies with a reference to Nancy. If it doesn't find one, it'll fall back to the `DefaultNancyBootstrapper`. You can only have **one** bootstrapper per application. If more than one custom bootstrapper is found in your application, Nancy tries to be smart and checks if one inherits from the other. When that is the case Nancy chooses the **most derived** bootstrapper.

This can be useful when you have several Nancy applications that all use the same base bootstrapper settings. But they have some bootstrapper setting specific for some of the applications.

**NOTE**: Sometimes, if you have class libraries (even with a reference to Nancy) where none of its types are used in the application at compile time, the compiler can be a bit *too* smart and removes the reference. For these scenarios it can be useful to add the `IncludeInNancyAssemblyScanningAttribute` to the class library (typically in `Properties.cs`):

    [assembly: IncludeInNancyAssemblyScanning]

This will make sure you have a *hard* reference to Nancy and the assembly will be included in Nancy scanning mechanism.

## Using AutoRegister

Part of the [Super-Duper-Happy-Path](https://github.com/NancyFx/Nancy/wiki/Introduction), when you use TinyIoC, is `AutoRegister`. Which allows you to piggyback on the IoC-Container, letting you define your own dependencies, that live next to Nancy's. For example injecting your own dependencies into a NancyModule.

```c#
public class Home : NancyModule
{
    public Home(IMessageService service)
    {
        //If there is only one implementation of IMessageService in the application,
        // TinyIoC will resolve the dependency on its own and inject it in the module.
    }
}
```

[<< Part 8. Model binding](Model binding) - [Documentation overview](Documentation) - [Part 10. View Engines >>](View Engines)