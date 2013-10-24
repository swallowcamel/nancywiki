Right smack in the center of Nancy lives this mythical unicorn that we like to call the `Bootstrapper`. Rather than being one monolithic framework, Nancy is actually lots of little pieces of functionality that are composed together when the application starts up. Configuration of this composition, including swapping out any of the default “pieces of the puzzle”, is done via the bootstrapper.

The bootstrapper is also responsible for some of the magic that you are treated to when working with the framework, such as automatic discovery of modules, custom model binders, dependencies and much more.

In essence the bootstrapper is a friendly syntax, or [Domain Specific Language (DSL)](http://en.wikipedia.org/wiki/Domain-specific_language), on top of an [Inversion of Control Container](http://en.wikipedia.org/wiki/Inversion_of_Control) that helps out with the component discovery and composition of the framework at runtime. Don’t worry if any of these terms sound new or scary, the super-duper-happy-path ensures that everything “just works” out of the box, without having to worry about the underlying container, and still allows very granular configuration/customisation as and when you require it.

The default implementation that is shipped with Nancy sits on top of the [TinyIoC](https://github.com/grumpydev/TinyIoC) IoC container; however, just as the bootstrapper lets you swap out pretty much any component in Nancy, the bootstrapper itself can be replaced with an implementation using a container of your choice. We ship implementations for all the major containers, available via [Nuget](https://nuget.org/packages?q=Nancy.Bootstrappers) or [our TeamCity server](http://teamcity.codebetter.com/project.html?projectId=project112&tab=projectOverview&guest=true) (for latest builds), but if there’s a container that’s not currently shipped that you think we should support - feel free to send a pull request :-)

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

## Ignoring assemblies when using AutoRegister.

When applications have many references it will take TinyIoC longer to scan through them to find implementations of the dependencies. To prevent this you can specify which assemblies Nancy can ignore. 

```c#
public class CustomBootstrapper : DefaultNancyBootstrapper
{
    protected override NancyInternalConfiguration InternalConfiguration
    {
        get
        {
            //This will tell Nancy it won't have to look in the Nhibernate or Lucene assemblies for implementations of your
            //interfaces.
            return NancyInternalConfiguration
                .Default
                .WithIgnoredAssembly (asm => asm.FullName.StartsWith ("NHibernate", StringComparison.InvariantCulture))
                .WithIgnoredAssembly (asm => asm.FullName.StartsWith ("Lucene", StringComparison.InvariantCulture));
        }
    }
}
```
When you do not use `AutoRegister` and do not plan to use it either you can also turn it off:

```c#
public class CustomBootstrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureApplicationContainer (TinyIoCContainer container)
    {
        //This can be achieved by not calling the "ConfigureApplicationContainer" base,
        //thus not configuring it to use AutoRegister.
        //base.ConfigureApplicationContainer (container);
    }
}
```
[<< Part 6. Model binding](Model binding) - [Documentation overview](Documentation) - [Part 8. View engines >>](View engines)