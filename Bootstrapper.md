Right smack in the center of Nancy lives this mythical unicorn that we like to call the `Bootstrapper`. Rather than being one monolithic framework, Nancy is actually lots of little pieces of functionality that are composed together when the application starts up. Configuration of this composition, including swapping out any of the default “pieces of the puzzle”, is done via the bootstrapper.

The bootstrapper is also responsible for some of the magic that you are treated to when working with the framework, such as automatic discovery of modules, custom model binders, dependencies and much more.

In essence the bootstrapper is a friendly syntax, or [Domain Specific Language (DSL)](http://en.wikipedia.org/wiki/Domain-specific_language), on top of a [Inversion of Control Container](http://en.wikipedia.org/wiki/Inversion_of_Control) that helps out with the component discovery and composition of the framework at runtime. Don’t worry if any of these terms sounds new or scary, the super-duper-happy-path ensures that everything “just works” out of the box, without having to worry about the underlying container, and still allows very granular configuration/customisation as and when you require it.

The default implementation that is shipped with Nancy sits on top of the [TinyIoC](https://github.com/grumpydev/TinyIoC) IoC container; however, just as the bootstrapper lets you swap out pretty much any component in Nancy, the bootstrapper itself can be replaced with an implementation using a container of your choice. We ship implementations for all the major containers, available via [Nuget](http://nuget.org) or [our TeamCity server](http://teamcity.codebetter.com/project.html?projectId=project112&tab=projectOverview&guest=true) (for latest builds), but if there’s a container that’s not currently shipped that you think we should support - feel free to send a pull request :-)

## Basic bootstrapper modifications

When you want to change the runtime behavior of Nancy you are going to be doing this though a custom bootstrapper. Fortunately this doesn’t mean you have to implement a bootstrapper from scratch each time, but instead you derive from the “base” bootstrapper you are using and override one of the methods / properties. In this example the ApplicationStartup method is overridden:

    public class CustomBootstrapper : DefaultNancyBootstrapper
    {
        public override void ApplicationStartup(TContainer container, IPipelines pipelines)
        {
            // your customizations goes here
        }
    }

## Adding a Custom Favicon

See http://en.wikipedia.org/wiki/Favicon for information on favicons.

To add a custom favicon embed a favicon.ico in your assembly and then override the NancyBootstrapperBase<TContainer>.DefaultFavIcon: 

    public class Bootstrapper:DefaultNancyBootstrapper
    {
        byte[] favicon;

        protected override byte[] DefaultFavIcon
        {
            get
            {
                if (favicon == null)
                {
                    //TODO: remember to replace 'AssemblyName' with the prefix of the resource
                    using (var resourceStream = GetType().Assembly.GetManifestResourceStream("AssemblyName.favicon.ico"))
                    {
                        var tempFavicon = new byte[resourceStream.Length];
                        resourceStream.Read(tempFavicon, 0, (int)resourceStream.Length);
                        favicon = tempFavicon;
                    }
                }
                return favicon;
            }
        }
    }


[<< Part 6. Model binding](Model binding) - [Documentation overview](Documentation) - [Part 8. View engines >>](View engines)