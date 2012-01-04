All paths that are used in Nancy are relative something that is known the _root path_. This is the path that tells Nancy where its resources are stored on the file system. The root path is provided to Nancy through an interface called the _IRootPathProvider_, which defines a single method _GetRootPath_.

The various hosting options are all shipped with their own implementation of this interface, because the process of figuring out where the application is located on the file system varies form host to host.

Nancy will automatically use the first implementation, of the _IRootPathProvider_, that it comes across so if you provide multiple implementations the end result will be non-deterministic. Care should be taken to ensure that only one implementation exists in the application domain of the running Nancy application.

## Changing the root path

Providing a custom root path is a two part process. First you need to create your own implementation of the _IRootPathProvider_ interface.

    public class CustomRootPathProvider : IRootPathProvider
    {
        public string GetRootPath()
        {
            return "What ever path you want to use as your application root";
        }
    }

Once that is done, the next thing you need to do is to let Nancy know that it should be using your own implementation, instead of going of and try to locate one for you (remember there are implementations in each host, so if you provide a second one, the result will be non-deterministic).

To let Nancy know it should use your implementation, you simply create a new bootstrapper (by inheriting from the bootstrapper you are using) and override the _RootPathProivider_ property and return the _type_ of your own root path provider.

    public class CustomBoostrapper : DefaultNancyBootstrapper
    {
        protected override Type RootPathProvider
        {
            get { return typeof(CustomRootPathProvider); }
        }
    }
When your application runs, the bootstrapper will register this implementation in the current container and will be used when dependencies on the _IRootPathProvider_ are resolved.


[<< Part 10. Testing your application](Testing your application) - [Documentation overview](Documentation) - [Part 11. Managing static content(Managing static content) >>]