## Ninject

First, install the [Nancy.Bootstrappers.Ninject](http://nuget.org/packages/Nancy.Bootstrappers.Ninject) package. Then, make your custom bootstrapper inherit from `NinjectNancyBootstrapper` instead of `DefaultNancyBootstrapper`. Finally, override the `ConfigureApplicationContainer` and the `ConfigureRequestContainer` methods, and bind your dependencies. The `container` parameter in `ConfigureRequestContainer` is a child container which is disposed at the end of the request.

    public class Bootstrapper : NinjectNancyBootstrapper
    {             
        protected override void ConfigureApplicationContainer(IKernel existingContainer)
        {
            //application singleton
            existingContainer.Bind<IApplicationSingleton>()
                .To<ApplicationSingleton>().InSingletonScope();
            //transient binding
            existingContainer.Bind<ICommandHandler>().To<CommandHandler>();
        }

        protected override void ConfigureRequestContainer(IKernel container, NancyContext context)
        {
            //container here is a child container. I.e. singletons here are in request scope.
            //IDisposables will get disposed at the end of the request when the child container does.
            container.Bind<IPerRequest>().To<PerRequest>().InSingletonScope();
        }
    }

## More to come

...