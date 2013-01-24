# Using SquishIt with Nancy

SquishIt is an asset processing project that helps speed up your Nancy website by combining and minifying your CSS, JavaScript, LESS and Coffeescript files.

https://github.com/jetheredge/SquishIt

You can read an introduction to SquishIt on the authors blog, here:

http://www.codethinked.com/squishit-the-friendly-aspnet-javascript-and-css-squisher

### Basic Setup

To use with NancyFX you first want to install the NuGet package to your project

`Install-Package SquishIt`

You create "bundles" of your asset files, as per the above introduction blog post.

Rather than create a Global.asax file in your project you do your wiring up of bundles in the ApplicationStart of your Nancy Bootstrapper.

### Advanced - Diskless caching

In release mode, SquishIt will write your minified files to disk. In shared hosting or in production this can cause problems or additional steps to your deployment scripts to assign folder permissions.

Following the approach listed here:

You can create an asset module to handle serving files from memory:

http://blogs.lessthandot.com/index.php/WebDev/UIDevelopment/AJAX/squishit-and-nancy

    public class AssetModule : NancyModule
    {
        public AssetModule() : base("/assets")
        {
            Get["/js/{name}"] = parameters => CreateResponse(Bundle.JavaScript().RenderCached((string)parameters.name), Configuration.Instance.JavascriptMimeType);

            Get["/css/{name}"] = parameters => CreateResponse(Bundle.Css().RenderCached((string)parameters.name), Configuration.Instance.CssMimeType);
        }

        Response CreateResponse(string content, string contentType)
        {
            return Response
                .FromStream(() => new MemoryStream(Encoding.UTF8.GetBytes(content)), contentType)
                .WithHeader("Cache-Control", "max-age=45");
        }
    }

### Advanced - Protecting your assets

In certain scenarios you don't want unauthenticated users to get access to all your assets. You would construct your asset module like:

    Get["/admin/js/{name}"] = parameters =>
    {
        this.RequiresAuthentication();
        this.RequiresClaims(new[] {"Admin"});

        return CreateResponse(Bundle.JavaScript().RenderCached((string) parameters.name), Configuration.Instance.JavascriptMimeType);
    };

    Get["/admin/css/{name}"] = parameters =>
    {
        this.RequiresAuthentication();
        this.RequiresClaims(new[] { "Admin" });

        return CreateResponse(Bundle.Css().RenderCached((string)parameters.name), Configuration.Instance.CssMimeType);
    };  

### Advanced - Testing with SquishIt

If you want to run tests against views with SquishIt used, you will most likely encounter:

`System.Collections.Generic.KeyNotFoundException : The given key was not present in the dictionary.`

With a stack trace that hints that something went bang in SquishIt. This is down to SquishIt not being able to find your asset files.

You can work around this by telling SquishIt to look in your full project rather than the bin of test project, for example:

	var fullProjectPath = "";

	var directoryName = Path.GetDirectoryName(typeof (Bootstrapper).Assembly.CodeBase);

	if (directoryName != null)
	{
	    var assemblyPath = directoryName.Replace(@"file:\", string.Empty);

	    fullProjectPath = Path.Combine(assemblyPath, "..", "..", "..", "Escape.Web");
	}

	bundle.Add(fullProjectPath + "/static/js/foo.js");

### Advanced - Only minifying certain files

If you are using something like AngularJS that relies on proper variable names for say dependancy injection, you can choose not to minify that particular file in SquishIt using:

`bundle.AddMinified("~/static/js/angular/controllers.js");`

### Advanced - The Full Monty

If you want to know how I've configured my application with SquishIt, here's how.

I defined a custom class to hold the url and if it should be minified or not (AngularJS doesn't like it so much):
```c#
    public class SquishItFile
    {
        public string Url { get; set; }

        public bool Minify { get; set; }
    }
```

I have a static class listing all my "to be bundles" assets:

```c#
    public static class Bundles
    {
        public static List<SquishItFile> PublicJavaScript = new List<SquishItFile>
        {
            new SquishItFile { Url = "~/public/static/js/jquery-1.6.4.js", Minify = true },
            new SquishItFile { Url = "~/public/static/js/startup.js", Minify = true }
        };

        public static List<SquishItFile> AdminJavaScript = new List<SquishItFile>
        {
            new SquishItFile { Url = "~/admin/static/js/lib/angular.js", Minify = true },
            new SquishItFile { Url = "~/admin/static/js/app/app.js", Minify = false }
        };

        public static List<SquishItFile> PublicCss = new List<SquishItFile>
        {
            new SquishItFile { Url = "~/public/static/css/site.css", Minify = true }
        };

        public static List<SquishItFile> AdminCss = new List<SquishItFile>
        {
            new SquishItFile { Url = "~/admin/static/less/bootstrap/bootstrap.less", Minify = true },
            new SquishItFile { Url = "~/admin/static/less/theme/theme.less", Minify = true }
        };
    }
```