# Hosting Nancy with ASP.Net

## Quick Start

Nancy under ASP.Net is handled through a HTTP Handler, which is setup through the Web.Config. If you're using one of Nancy's Visual Studio templates then the configuration is handler for you, if not then you'll need to add the following to configuration section of your Web.Config:

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

Also make sure you have referenced not only the Nancy project/assembly, but also the Nancy.Hosting.Aspnet project/assembly.

That's all you need to do to get up and running.

**Special note about IIS 6 and PUT/DELETE requests:** By default IIS 6 does not support PUT and DELETE verbs. To enable this, you need to add a wildcard mapping to the virtual directory of your Nancy application - read the "IIS6 Extension-less URLs" section in this document: http://haacked.com/archive/2008/11/26/asp.net-mvc-on-iis-6-walkthrough.aspx



## Bootstrapping

By default, the Nancy HTTP Handler will use the built in "Bootstrapper Locator" for identifying the best bootstrapper to kick-start Nancy. This behaviour should be fine for most scenarios; but if you want to take control over which bootstrapper is used, you can do so using the Web.Config:

    <configSections>
      <section name="nancyFx" type="Nancy.Hosting.NancyFxSection" />
    </configSections>
    <nancyFx>
      <bootstrapper assembly="Nancy.Demo" type="Nancy.Demo.DemoBootStrapper"/>
    </nancyFx>

Here we define a NancyFx configuration section and a bootstrapper entry that specified the type of the bootstrapper, and the assembly it's located in. If this configuration setting is specified then the locator is bypassed completely and Nancy will use the specified type instead. 

For more information on Nancy Bootstrapping please see [[Bootstrapping-Nancy]].

## Adding Nancy to an existing site

Sometimes you have an existing ASP.NET site and you want to configure Nancy to handle requests to a particular path. To do this, follow the instructions above but set `path="nancypath/*"` on the `NancyHttpRequestHandler` handlers. You will also need to add a project folder to represent the path, and in that folder you would add the following Web.config:

    <?xml version="1.0"?>
    <configuration>
        <system.web>
          <httpHandlers>
            <add verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="*"/>
          </httpHandlers>
        </system.web>
    </configuration>
