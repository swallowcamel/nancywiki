## Ninject

First, download the Nancy.Bootstrappers.Ninject package. Then, make your custom bootstrapper inherit from `NinjectNancyBootstrapper` instead of `DefaultNancyBootstrapper`. And finally, override the `ConfigureRequestContainer` method, and bind your dependencies.

    public class Bootstrapper : NinjectNancyBootstrapper
    {             
        protected override void ConfigureRequestContainer(
             IKernel container, NancyContext context)
        {
            container.Bind<ICommandHandler>().To<CommandHandler>();
        }    
    }

## More to come

...