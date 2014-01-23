In case you want to host your Nancy application while using a web server of your choice, be it IIS, Apache, Nginx, Lighttpd or any other FastCgi enabled web server, there is the option of using [Fos](http://github.com/mzabani/Fos).

**Fos** is a library that allows an Owin console application to receive requests through a FastCgi socket from a web server. **Fos** is also designed to run well with Mono on all platforms.

## Setting up your application with nginx on Linux

We will show you how to set up FastCgi on *nginx* and how to run your Owin application with Fos. The idea applies to any web server, so feel free to try others.

First, let us configure *nginx* by editing */etc/nginx/nginx.conf* (or wherever your distribution put that file), and modifying your main `location` block like shown below:

```
server {
    listen 80;
    server_name localhost;
    ...
    
    location / {
        fastcgi_pass 127.0.0.1:9000;
        include /etc/nginx/fastcgi_params;
    }
}
```

That's it for *nginx*! Now let's create a *Nancy* application to run it with **Fos**. Start a new **console project** and install *Nancy* and *Nancy.Owin* (via NuGet, for example). After that, change your application's *Main* method's code and your main class to something in the lines of:

```c#
using Fos;
using Fos.Owin;
using Nancy;
using Nancy.Owin;

namespace MyApplication.Startup {
    private static void ConfigureOwin(IAppBuilder builder) {
        builder.UseNancy();
    }

    public static void Main(string[] args) {
        using (var fosServer = new FosSelfHost(ConfigureOwin))
        {
            fosServer.Bind(System.Net.IPAddress.Loopback, 9000);
            fosServer.Start(false);
        }
    }
}
```

And that is it! Don't forget to either restart *nginx* or reload the configuration file. Then just build and run your application and you're good to go!
You can also start **Fos** with a Unix Socket, not to mention it comes with some Owin middleware to help you out with logging access times and some other stuff. Check the [official website](http://github.com/mzabani/Fos) for more.

## More info

You can (and **should**, if you want to use Fos) read more about **Fos** at the [official website](http://github.com/mzabani/Fos). **Fos** is still in its infancy so you should probably not use it for production.
Also, if anyone knows how to set up IIS with FastCgi through a TCP Socket, the author of **Fos** would very much like to know how to put that information up.