# What is OWIN?
OWIN defines a standard interface between .NET web servers and web applications. The goal of the OWIN interface is to decouple server and application, encourage the development of simple modules for .NET web development, and, by being an open standard, stimulate the open source ecosystem of .NET web development tools. For more information please visit the official website at http://owin.org/

# Katana
[Katana](http://katanaproject.codeplex.com/documentation) is OWIN implementations for Microsoft servers and frameworks.

## Katana - ASP.NET Host
*Note: Assumes your project is an ASP.NET project with Nancy.dll referenced and without any Nancy.Hosting.xxxxx.dll referenced.*

* **Install package using Nuget**
```
Install-Package Microsoft.Owin.Host.SystemWeb
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