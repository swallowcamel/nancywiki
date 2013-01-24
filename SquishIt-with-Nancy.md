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

### Advanced

The following sections (with the last being a full code example) cover advanced scenarios and topics of SquishIt. The specific code around asset location in unit testing is specific to Nancy on .NET and Windows and will most likely not work on Mono.

### Advanced - Diskless caching

In release mode, SquishIt will write your minified files to disk. In shared hosting or in production this can cause problems or additional steps to your deployment scripts to assign folder permissions.

Following the approach listed here:

You can create an asset module to handle serving files from memory:

http://blogs.lessthandot.com/index.php/WebDev/UIDevelopment/AJAX/squishit-and-nancy

```c#
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
```

### Advanced - Protecting your assets

In certain scenarios you don't want unauthenticated users to get access to all your assets. You would construct your asset module like:

```c#
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
```

### Advanced - Testing with SquishIt

If you want to run tests against views with SquishIt used, you will most likely encounter:

```c#
System.Collections.Generic.KeyNotFoundException : The given key was not present in the dictionary.
```

With a stack trace that hints that something went bang in SquishIt. This is down to SquishIt not being able to find your asset files.

You can work around this by telling SquishIt to look in your full project rather than the bin of test project, for example:

```c#
	var fullProjectPath = "";

	var directoryName = Path.GetDirectoryName(typeof (Bootstrapper).Assembly.CodeBase);

	if (directoryName != null)
	{
	    var assemblyPath = directoryName.Replace(@"file:\\", string.Empty);

	    fullProjectPath = Path.Combine(assemblyPath, "..", "..", "..", "Escape.Web");
	}

	bundle.Add(fullProjectPath + "/static/js/foo.js");
```

### Advanced - Only minifying certain files

If you are using something like AngularJS that relies on proper variable names for say dependancy injection, you can choose not to minify that particular file in SquishIt using:

```c#
bundle.AddMinified("~/static/js/angular/controllers.js");
```

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

I have a "startup" class that handles building the bundles from my lists of files

```c#
    public class SquishItStartup
    {
        protected static string BasePathForTesting = "";

        protected static JavaScriptBundle BuildJavaScriptBundle(List<SquishItFile> files)
        {
            var bundle = Bundle.JavaScript();

            foreach (var item in files)
            {
                var url = item.Url;

                if (!string.IsNullOrWhiteSpace(BasePathForTesting))
                {
                    url = BasePathForTesting + item.Url.Replace("~", "");
                }

                if (item.Minify)
                {
                    bundle.Add(url);
                }
                else
                {
                    bundle.AddMinified(url);
                }
            }

            return bundle;
        }

        protected static CSSBundle BuildCssBundle(List<SquishItFile> files)
        {
            var bundle = Bundle.Css();

            foreach (var item in files)
            {
                var url = item.Url;

                if (!string.IsNullOrWhiteSpace(BasePathForTesting))
                {
                    url = BasePathForTesting + item.Url.Replace("~", "");
                }

                if (item.Minify)
                {
                    bundle.Add(url);
                }
                else
                {
                    bundle.AddMinified(url);
                }
            }

            return bundle;
        }

        public static void Setup(string basePathForTesting = "")
        {
            BasePathForTesting = basePathForTesting;

            // CSS
            BuildCssBundle(Bundles.PublicCss).ForceRelease().AsCached("public-css", "~/assets/css/public-css");
            BuildCssBundle(Bundles.PublicCss).ForceDebug().AsNamed("public-css-debug", "");

            BuildCssBundle(Bundles.AdminCss).ForceRelease().AsCached("admin-css", "~/assets/admin/css/admin-css");
            BuildCssBundle(Bundles.AdminCss).ForceDebug().AsNamed("admin-css-debug", "");

            // JS
            BuildJavaScriptBundle(Bundles.PublicJavaScript).ForceRelease().AsCached("public-js", "~/assets/js/public-js");
            BuildJavaScriptBundle(Bundles.PublicJavaScript).ForceDebug().AsNamed("public-js-debug", "");

            BuildJavaScriptBundle(Bundles.AdminJavaScript).ForceRelease().AsCached("admin-js", "~/assets/admin/js/admin-js");

            BuildJavaScriptBundle(Bundles.AdminJavaScript).ForceDebug().AsNamed("admin-js-debug", "");
        }
    }
```

This is then used in my custom Nancy Boostrapper like so:

```c#
   public class Bootstrapper : DefaultNancyBootstrapper
    {
        protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines)
        {
            base.ApplicationStartup(container, pipelines);

            // Setup SquishIt bundles
            SquishItStartup.Setup();
        }
    }
```

Then in my Razor layout view file I can call SquishIt like:

```c#
    @if (StaticConfiguration.IsRunningDebug)
    {
        @Html.Raw(Bundle.Css().RenderNamed("public-css-debug"))
    }
    else
    {
        @Html.Raw(Bundle.Css().RenderCachedAssetTag("public-css"))
    }

    @if (StaticConfiguration.IsRunningDebug)
    {
        @Html.Raw(Bundle.JavaScript().RenderNamed("public-js-debug"))
    }
    else
    {
        @Html.Raw(Bundle.JavaScript().RenderCachedAssetTag("public-js"))
    }
```

This enabled me to have faster debugging as it doesn't process files but get the benefits when I publish my website.

And in my custom Nancy testing Bootstrapper so SquishIt can find my asset files:

```c#
    public class TestingNancyBootstrapper : Bootstrapper
    {
        protected string PathToProjectWebRoot()
        {
            var directoryName = Path.GetDirectoryName(typeof (Bootstrapper).Assembly.CodeBase);

            if (directoryName != null)
            {
                var assemblyPath = directoryName.Replace(@"file:\\", string.Empty);

                return Path.Combine(assemblyPath, "..", "..", "..", "MyProject.Web");
            }

            return "";
        }

        protected override void ApplicationStartup(TinyIoCContainer container, Nancy.Bootstrapper.IPipelines pipelines)
        {
            // Setup SquishIt bundles
            SquishItStartup.Setup(PathToProjectWebRoot());
        }
    }
```