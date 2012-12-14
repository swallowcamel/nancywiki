As a default, Nancy will use an icon, of the Nancy logo, as the favicon for any Nancy application unless you provider you own. Overriding this behavior, to provide your own custom icon, is very simple

## Replacing the default FavIcon
Simple place a `.ico` or `.png` file, called _favicon_ anywhere in your application path (to learn more about the application root path, please consult the [[The Root Path]] section) and Nancy will recursively scan your application, at start up, for the file.

If you have more then one favicon in your application, the first one it finds will be used.

### Using an embedded icon
You can also embed a favicon in your application assembly. To make Nancy use the embedded icon, simply override the `DefaultFavIcon` property, of your bootstrapper, and add the following code 

```c#
public class Bootstrapper : DefaultNancyBootstrapper
{
    private byte[] favicon;

    protected override byte[] FavIcon
    {
        get { return this.favicon?? (this.favicon= LoadFavIcon()); }
    }
}

private static byte[] LoadFavIcon()
{
    //TODO: remember to replace 'AssemblyName' with the prefix of the resource
    using (var resourceStream = GetType().Assembly.GetManifestResourceStream("AssemblyName.favicon.ico"))
    {
        var tempFavicon = new byte[resourceStream.Length];
        resourceStream.Read(tempFavicon, 0, (int)resourceStream.Length);
        return tempFavicon;
    }
}
```

### Removing the icon

If you do not wish to use a favicon, at all, then simply override the `FavIcon` property, on your [[Bootstrapper]] and return `null`

### Help! I can't see my icon

I bet if you clear your browsers cache you will see it!


[<< Part 13. Diagnostics](Diagnostics) - [Documentation overview](Documentation) - [Part 15. The Application Before, After and OnError pipelines >>](The Application Before, After and OnError pipelines)