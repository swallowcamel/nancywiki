By default, Nancy will use an icon of the Nancy logo as the favicon for any Nancy application, unless you provide you own. Overriding this behavior to provide your own custom icon is very simple

## Replacing the default FavIcon
Simply place a `.ico` or `.png` file, called _favicon_ anywhere in your application path (to learn more about the application root path, please consult the [[The Root Path]] section) and Nancy will recursively scan your application, at start up, for the file.

If you have more then one favicon in your application, the first one it finds will be used.

### Using an embedded icon
You can also embed a favicon in your application assembly. To make Nancy use the embedded icon, simply override the `FavIcon` property of your bootstrapper, and add the following code:

```c#
public class Bootstrapper : DefaultNancyBootstrapper
{
    private byte[] favicon;

    protected override byte[] FavIcon
    {
        get { return this.favicon?? (this.favicon = LoadFavIcon()); }
    }

    private byte[] LoadFavIcon()
    {
        //TODO: remember to replace 'AssemblyName' with the prefix of the resource
        using (var resourceStream = GetType().Assembly.GetManifestResourceStream("AssemblyName.favicon.ico"))
        {
            var tempFavicon = new byte[resourceStream.Length];
            resourceStream.Read(tempFavicon, 0, (int)resourceStream.Length);
            return tempFavicon;
        }
    }
}
```

### Removing the icon

If you do not wish to use a favicon at all, then simply override the `FavIcon` property on your [[Bootstrapper]] and return `null`.

Note that returning `null` will result in requests for `/favicon.ico` being passed along the pipeline, which means (in particular) that your modules may see this request.

### Help! I can't see my icon

Try:

1. Clearing your browser's cache.
2. Navigating to `http://site/favicon.ico` and forcing a refresh (usually Ctrl+F5).

[<< Part 16. Diagnostics](Diagnostics) - [Documentation overview](Documentation) - [Part 18. Generating a custom error page >>](Generating a custom error page)