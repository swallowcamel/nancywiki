# Quick Start

Nancy under ASP.Net is handled through a HTTP Handler, which is setup through the Web.Config. If you're using one of Nancy's Visual Studio templates then the configuration is handled for you, if not then you'll need to add the following to the configuration section of your Web.Config:

```xml
<system.web>
  <compilation debug="true" targetFramework="4.0" />
  <httpHandlers>
    <add verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="*"/>
  </httpHandlers>
</system.web>

<system.webServer>
  <modules runAllManagedModulesForAllRequests="true"/>
  <validation validateIntegratedModeConfiguration="false"/>
  <handlers>
    <add name="Nancy" verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="*"/>
  </handlers>
</system.webServer>
```
Also make sure you have referenced not only the Nancy project/assembly, but also the Nancy.Hosting.Aspnet project/assembly.

That's all you need to do to get up and running.

**Special note about PUT/DELETE requests:** 

By default IIS 6 does not support PUT and DELETE verbs. To enable this, you need to add a wildcard mapping to the virtual directory of your Nancy application, this is as simple as clicking the insert button next to Wildcard application maps in your site or virtual directory properties in IIS Manager and entering the path to aspnet_isapi.dll. The Wildcard mappings are accessible on the home directory (or virtual directory) tab via the configuration button.   The simplest and safest way to get the path to your aspnet_isapi.dll is to copy it from the application extension mapping for the .aspx extension displayed on the same tab.  This is much the same as the option for getting asp.net mvc handling iis requests described in the "IIS6 Extension-less URLs" section in this document: http://haacked.com/archive/2008/11/26/asp.net-mvc-on-iis-6-walkthrough.aspx

You might receive "405 Not allowed" pages while trying to make PUT/DELETE requests on IIS 7/7.5.
One way to fix it is to remove the WebDAVModule in the web.config. 

```xml
<system.webServer>
  <modules runAllManagedModulesForAllRequests="true">
    <remove name="WebDAVModule" />
  </modules>  
</system.webServer>
```
## Bootstrapping

By default, the Nancy HTTP Handler will use the built in "Bootstrapper Locator" for identifying the best bootstrapper to kick-start Nancy. This behaviour should be fine for most scenarios; but if you want to take control over which bootstrapper is used, you can do so using the Web.Config:

```xml
<configSections>
  <section name="nancyFx" type="Nancy.Hosting.Aspnet.NancyFxSection" />
</configSections>
<nancyFx>
  <bootstrapper assembly="Nancy.Demo" type="Nancy.Demo.DemoBootStrapper"/>
</nancyFx>
```
Here we define a NancyFx configuration section and a bootstrapper entry that specified the type of the bootstrapper, and the assembly it's located in. If this configuration setting is specified then the locator is bypassed completely and Nancy will use the specified type instead. 

For more information on Nancy Bootstrapping please see [[Bootstrapping-Nancy]].

## Adding Nancy to an existing site

Sometimes you have an existing ASP.NET site and you want to configure Nancy to handle requests to a particular path. To do this, follow the instructions above but set `path="nancypath/*"` on the `NancyHttpRequestHandler` handlers. You will also need to add a project folder to represent the path, and in that folder you would add the following Web.config:

```xml
<?xml version="1.0"?>
<configuration>
  <system.web>
    <httpHandlers>
      <add verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="*"/>
    </httpHandlers>
  </system.web>
</configuration>
```
Alternatively, enclose the setting within `<location>` in your root web.config:

```xml
<location path="nancy">
  <system.web>
    <compilation debug="true" targetFramework="4.0" />
    <httpHandlers>
      <add verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="*"/>
    </httpHandlers>
  </system.web>

  <system.webServer>
    <modules runAllManagedModulesForAllRequests="true"/>
    <validation validateIntegratedModeConfiguration="false"/>
    <handlers>
      <add name="Nancy" verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="*"/>
    </handlers>
  </system.webServer>
</location>
```
Note that your modules will still respond to the full path including the location, ie ["/nancy/.."]

## Adding Nancy to an existing ASP.Net MVC site

Follow all the instructions above but make sure to remove the "nancy" path from the RoutingTable (full story [read here](http://igorshare.wordpress.com/2012/04/08/adding-nancy-to-the-existing-asp-net-mvc-site/)).
Add the following instruction to the Global.asax.cs file inside the MvcApplication.RegisterRoutes:

```c#
routes.IgnoreRoute("nancy/{*pathInfo}");
```

It is worth noting that the path that you setup for Nancy, in the routing table, will be part of the requested path to your Nancy module. Using the route above, that means if you wanted to access a `/products` resource, the route would have to be declared for `/nancy/products`

```c#
public class Home : NancyModule
{
   public Home()
   {
      Get["/nancy/products"] = x => "The products!";
   }
}
```

You can make use of the base path, of the module, to make it cleaner

```c#
public class Home : NancyModule
{
   public Home() : base("/nancy")
   {
      Get["/products"] = x => "The products!";
   }
}
```

## Prevent IIS from taking over errors

This is added by default since 0.22.

```c#
<system.webServer>
    <httpErrors existingResponse="PassThrough"/>
</system.webServer>
```

On IIS7 you aren't allowed to change this value on the website level. You end up with one line errors like _"The page cannot be displayed because an internal server error has occurred."_. You can fix this by running this as administrator: `%windir%\system32\inetsrv\appcmd.exe unlock config -section:system.webServer/httpErrors`. 