# What is OWIN?
OWIN defines a standard interface between .NET web servers and web applications. The goal of the OWIN interface is to decouple server and application, encourage the development of simple modules for .NET web development, and, by being an open standard, stimulate the open source ecosystem of .NET web development tools. For more information please visit the official website at http://owin.org/

# Katana
[Katana](http://katanaproject.codeplex.com/documentation) is OWIN implementations for Microsoft servers and frameworks.

## Katana - ASP.NET Host
*Note: Assumes your project is an ASP.NET project with Nancy.dll referenced and without any Nancy.Hosting.xxxxx.dll referenced.*

* **Install package using Nuget**
```
Install-Package Microsoft.Owin.Host.SystemWeb
Install-Package Nancy.Owin
```

* **Add the following key in `web.config`**. (This is required for the current v1.0.1 for SystemWeb OWIN host and is likely to be removed in future versions.)
```
<appSettings>
    <add key="owin:HandleAllRequests" value="true"/>
</appSettings>
```
* **Create an OWIN startup file**
```c#
using Owin;

public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseNancy();
    }
}
```
*Note: You must keep the Startup file's namespace the same as your application name or it will return a `403.14 - Forbidden`.*

## Katana - HttpListener (SelfHost)

* **Install packages using NuGet**
```
Install-Package Microsoft.Owin.Hosting -pre
Install-Package Microsoft.Owin.Host.HttpListener -pre
Install-Package Nancy.Owin
```
*Note: As of this writing `Microsoft.Owin.Hosting` and `Microsoft.Owin.Host.HttpListener` host is in preview and is not recommended to be used in production. Make sure to add `-pre` to install.*
* **Create an OWIN startup file**
```c#
using Owin;

public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseNancy();
    }
}
```
* **Main Entry Point**
```c#
using Microsoft.Owin.Hosting;

class Program
{
    static void Main(string[] args)
    {
        var url = "http://+:8080";

        using (WebApp.Start<Startup>(url))
        {
            Console.WriteLine("Running on http://localhost:8080", url);
            Console.WriteLine("Press enter to exit");
            Console.ReadLine();
        }
    }
}
```
* Run Visual Studio or the compiled executable in admin mode.

**Running without Admin mode**

Note that on Windows hosts a `HttpListenerException` may be thrown with an `Access Denied` message. To resolve this the URL has to be added to the ACL. 

On Windows Vista/Server 2008 or later, execute the following in PowerShell or CMD running as administrator:

    netsh http add urlacl url=http://+:8080/ user=DOMAIN\username

Replace `DOMAIN\username` with your domain and username or your computer name and username if you are not joined to a domain. See <http://msdn.microsoft.com/en-us/library/ms733768.aspx> for more information. You can find the user with `whoami` command.

You can remove the urlacl using the following command.

    netsh http delete urlacl url=http://+:8080/

Also but the port may need to be opened on the machine or corporate firewall to allow access to the service.

# Accessing OWIN environment variables

```c#
public class HomeModule : NancyModule
{
    public HomeModule()
    {
      Get["/"] = x => {
            var env                 = (IDictionary<string, object>)Context.Items[NancyOwinHost.RequestEnvironmentKey];

            var requestBody         = (Stream)env["owin.RequestBody"];
            var requestHeaders      = (IDictionary<string, string[]>)env["owin.RequestHeaders"];
            var requestMethod       = (string)env["owin.RequestMethod"];
            var requestPath         = (string)env["owin.RequestPath"];
            var requestPathBase     = (string)env["owin.RequestPathBase"];
            var requestProtocol     = (string)env["owin.RequestProtocol"];
            var requestQueryString  = (string)env["owin.RequestQueryString"];
            var requestScheme       = (string)env["owin.RequestScheme"];

            var responseBody        = (Stream)env["owin.ResponseBody"];
            var responseHeaders     = (IDictionary<string, string[]>)env["owin.ResponseHeaders"];

            var owinVersion         = (string)env["owin.Version"];
            var cancellationToken   = (CancellationToken)env["owin.CallCancelled"];

            var uri = (string)env["owin.RequestScheme"] + "://" + requestHeaders["Host"].First() +
              (string)env["owin.RequestPathBase"] + (string)env["owin.RequestPath"];

            if (env["owin.RequestQueryString"] != "")
              uri += "?" + (string)env["owin.RequestQueryString"];

            return string.Format("{0} {1}", requestMethod, uri);
      };
    }
}
```