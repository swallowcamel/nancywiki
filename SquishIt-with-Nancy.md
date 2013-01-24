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

### Advanced - Only minifying certain files

If you are using something like AngularJS that relies on proper variable names for say dependancy injection, you can choose not to minify that particular file in SquishIt using:

`bundle.AddMinified("~/static/js/angular/controllers.js");`