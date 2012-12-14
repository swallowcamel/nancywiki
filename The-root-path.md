All paths that are used in Nancy are relative something that is known the _root path_. This is the path that tells Nancy where its resources are stored on the file system. The root path is provided to Nancy through an interface called the _IRootPathProvider_, which defines a single method _GetRootPath_.

The various hosting options are all shipped with their own implementation of this interface, because the process of figuring out where the application is located on the file system varies from host to host.

Nancy will automatically use the first implementation, of the _IRootPathProvider_, that it comes across so if you provide multiple implementations the end result will be non-deterministic. Care should be taken to ensure that only one implementation exists in the application domain of the running Nancy application.

## Changing the root path

Providing a custom root path is a two part process. First you need to create your own implementation of the `IRootPathProvider` interface.
```c#
public class CustomRootPathProvider : IRootPathProvider
{
    public string GetRootPath()
    {
        return "What ever path you want to use as your application root";
    }
}
```

Once that is done, the next thing you need to do is to let Nancy know that it should be using your own implementation, instead of going of and try to locate one for you (remember there are implementations in each host, so if you provide a second one, the result will be non-deterministic).

To let Nancy know it should use your implementation, you simply create a new bootstrapper (by inheriting from the bootstrapper you are using) and override the _RootPathProivider_ property and return the _type_ of your own root path provider.

```c#
public class CustomBootstrapper : DefaultNancyBootstrapper
{
    protected override Type RootPathProvider
    {
        get { return typeof(CustomRootPathProvider); }
    }
}
```
When your application runs, the bootstrapper will register this implementation in the current container and will be used when dependencies on the `IRootPathProvider` are resolved.

##Uploading Files

To upload a file in Nancy you need to take the content stream of the uploaded file, create a file on disk and write that stream to disk.

    var uploadDirectory = pathProvider.GetRootPath() + "\\Content\\uploads";
    
    if (!Directory.Exists(uploadDirectory))
    {
      Directory.CreateDirectory(uploadDirectory);
    }
    
    foreach (var file in Request.Files)
    {
      using (FileStream fileStream = System.IO.File.Create(Path.Combine(uploadDirectory, file.Name), (int)file.Value.Length))
      {
         // Fill the bytes[] array with the stream data
         byte[] bytesInStream = new byte[file.Value.Length];
         file.Value.Read(bytesInStream, 0, (int)bytesInStream.Length);
    
         // Use FileStream object to write to the specified file
         fileStream.Write(bytesInStream, 0, bytesInStream.Length);
      }
    }

However, you may be wondering what is pathProvider. This variable is passed into our module constructor which gives us access to the root path of our application by calling `GetRootPath()` and then you can save to a folder within your application.

    public HomeModule(IRootPathProvider pathProvider)


[<< Part 10. Testing your application](Testing your application) - [Documentation overview](Documentation) - [Part 12. Managing static content](Managing static content) >>