# Bootstrapping Nancy #

## These Boots Are Made For Walkin'.. ##

Although you can happily construct NancyEngine yourself, and inject dependencies however you see fit, in normal use you will be taking advantage of a Bootstrapper that will:

* Wireup Nancy's internal dependencies for you.
* Allow you to customise just about everything - how much control you want is entirely up to you.

## The Super-Duper-Happy-Path ##

Although the exact instructions may vary slightly depending on your choice of [[Hosting]], all you generally have to do is add a NancyModule to your project. That's it. No configuration, no registering dependencies, just 100% pure Nancy goodness - if your module has dependencies Nancy will even wire those up too!

For example, if you add this module to your project:

    public class MyModule : NancyModule
    {
      private IMyDependency dependency;
      
      public MyModule(IMyDependency dependency)
      {
          this.dependency = dependency;
          
          Get["/"] = x => {
          };
          
          // Register other routes
      }
    }

Nancy will automatically discover your module and, assuming you have something that implements IMyDependency, it will automatically be constructed and injected into your module - all without you lifing a finger!

We did say it was a Super-Duper-Happy-Path!

## Taking Control ##

### Part 1 - Customisation Options ###

Nancy ships with a "default bootstrapper" that takes care of all the magic in the above example. If you add your own Bootstrapper to your project, derived from DefaultNancyBootstrapper, Nancy will automatically detect it and use it in place of the default one. 

_Note: Depending on your hosting choice you may also be able to override the Bootstrapper in alternative ways such as via the Web.Config in ASP.Net applications._

Once you have your own Bootstrapper class you can replace some or all of the default behaviour by overriding properties and methods. The following overrides are available:

* _**Default... Properties**_ - These properties define the implementations of Nancy's internal interfaces that Nancy registeres into the container. There shouldn't be a need to switch any of these out, but the option is there should you want it :-)
* _**GetModuleTypes**_ - The default implementation of this scans the application domain for NancyModules for registration into the container. Each module that is discovered is given a unique "key" so they can be retrieved individually from the container when satisfying a request.
* _**ConfigureApplicationContainer**_ / **_ConfigureRequestContainer_** - manually register dependencies - see Part 2 below for more details. 

### Part 2 - Manually Registering Dependencies ###

By default Nancy uses an embedded Inversion of Control container called "[TinyIoC](http://hg.grumpydev.com/tinyioc)" ( see <http://hg.grumpydev.com/tinyioc> for more information ). The built in Bootstrapper uses [TinyIoC](http://hg.grumpydev.com/tinyioc) to scan types on application startup and registers them into the container to automatically resolve dependencies.

If you don't want to take advantage of the "AutoRegister" feature you can take control of adding dependencies into the container by overriding one or both of the following methods:

* _**ConfigureApplicationContainer**_ - This is called once, when the Bootstrapper is executed, and is used to register dependencies that you either wish to have application scope lifetimes (application singletons), or be registered as multi-instance:
* _**ConfigureRequestContainer**_ - This is called once per request, before the module matching the route is resolved, and is used to register singletons that will have request lifetime. 

Both of these methods are passed the container instance as a parameter:

    container.Register<IMyInterface, MyImplementation>().AsSingleton();
    container.Register<IMyInterfaceToo, MyOtherThing>().AsMultiInstance();
    
For more information on the various registration options available, please take a look at the [TinyIoC Wiki](<http://hg.grumpydev.com/tinyioc>).

### Part 3 - Alternative Containers ###

Although they don't form part of the "core" Nancy project, Nancy's Bootstrapper architecture is designed to be easily adapted to other IoC containers. To use an alternative Bootstrapper simply derive from the replacement Bootstrapper, rather than the default Nancy one, and Nancy will take care of the rest.

For a list of alternative container implementations please see [[Container Support]].

### Part 4 - Barebones Hardcore Ninja Mode / Being Flippin' Awkward ###

Although Nancy provides the Bootstrapper base classes to provide a simple, customisable, and consistent interface; the only requirement for a Bootstrapper is that it implements INancyBootstrapper:

    /// <summary>
    /// Bootstrapper for the Nancy Engine
    /// </summary>
    public interface INancyBootstrapper
    {
        /// <summary>
        /// Gets the configured INancyEngine
        /// </summary>
        /// <returns>Configured INancyEngine</returns>
        INancyEngine GetEngine();
    }