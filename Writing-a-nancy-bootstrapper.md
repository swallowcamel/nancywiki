# Writing a Nancy BootStrapper
## General Information

Bootstrappers should be implemented as **abstract classes** - the idea being that the end users inherit from a specific bootstrapper implementation and override whatever they need to (if anything) to get things configured as they want.

It is recommended that any virtual/abstract methods from NancyBootStrapperBase are sealed in the bootstrapper implementation where it doesn't make sense for the user to override them. This is simply to cut down on clutter and to try and give us the largest opening to the pit of success :-) The DefaultNancyBootStrapper does this.

## Application Lifecyle / Lifetimes

If you are implementing everything there are several "stages" to initialising the container. Firstly, the stuff that happens at application startup:

* **ConfigureApplicationContainer** - this is where users register their bits at the application level - they're free to choose whatever lifetime they want.
* **RegisterDefaults** - gets passed a list of types that Nancy itself needs. These should be registered as application lifetime singletons.
* **RegisterModules** - gets passed a list of modules. These should be registered as multi-instance OR per-request lifetime (see below).
* **GetAllModules** - this is only called on application startup, but if you're implementing child containers for per-request lifetimes (see below), then this should also configure the child container before resolving using it.

The following is called per-request:

* **GetModuleByKey** - retrieve a single module by its key. If you're implementing child containers then this should configure the child container before resolving using it.

## Child Containers

Most modern containers support ASP.NET per-request lifetimes, which is great, but as Nancy is intended to run on things other than ASP.NET, that can't be the only solution. To this end we have a separate interface, `INancyBootStrapperPerRequestRegistration<T>` (although it might get renamed ;-)). This interface provides a single method called `ConfigureRequestContainer()`, which should be implemented as a virtual method in the Bootstrapper, that allows the user to register any dependencies that should only live for the lifetime of that request. Now, when `GetAllModules()` or `GetModuleByKey()` is called, the workflow should be:

Get a child container from the main container -> pass child container to `ConfigureRequestContainer` -> user registers types (usually singletons like UoW) -> Bootstrapper uses the child container to resolve the module(s)
