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

### Advanced - Protecting your assets

In certain scenarios you don't want un-authenticated users to get access to all your assets. You would construct your asset module like:

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

