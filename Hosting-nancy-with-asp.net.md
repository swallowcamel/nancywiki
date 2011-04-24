# Hosting Nancy with ASP.Net

## Quick Start

Nancy under ASP.Net is handled through a HTTP Handler, which is setup through the Web.Config. If you're using one of Nancy's Visual Studio templates then the configuration is handler for you, if not then you'll need to add the following to configuration section of your Web.Config:

    <system.web>
      <compilation debug="true" targetFramework="4.0" />
      <httpHandlers>
        <add verb="*" type="Nancy.Hosting.NancyHttpRequestHandler" path="*"/>
      </httpHandlers>
    </system.web>

    <system.webServer>
       <modules runAllManagedModulesForAllRequests="true"/>
      <validation validateIntegratedModeConfiguration="false"/>
      <handlers>
        <add name="Nancy" verb="*" type="Nancy.Hosting.NancyHttpRequestHandler" path="*"/>
      </handlers>
    </system.webServer>

That's all you need to do to get up and running.

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