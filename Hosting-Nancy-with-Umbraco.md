These are simple instructions for hosting a service within an existing [Umbraco](http://umbraco.com/) installation.

## Umbraco Is Just ASP.Net...Sort of
The key is to remember that Umbraco is just another ASP.Net application and as such the "Adding Nancy to an existing site" section in [[Hosting-nancy-with-asp.net]] will get you most of the way there.

The Umbraco specific step is to edit the Web.config file to make sure Umbraco ignores requests to our service path. This is simply a matter of adding the path to the umbracoReservedPaths configuration option. For example:
`
<add key="umbracoReservedPaths" value="~/umbraco,~/install/,~/services" />
`

## Quick Example
Let's assume we are implementing a set of JSON services to consume from our Umbraco pages and we want to put our set of services under `/services` so that we can make requests such as `/services/vegetables` to get a list of all available vegetables.

### Our Service Implementation
Our simple service implementation might look like this:

```c#
namespace vegetablesrock.com.services
{
    using Nancy;
    using vegetablesrock.com.common;
    public class VegetableServices : NancyModule
    {
        public VegetableServices(): base("services")
        {
            Get["/Vegetables"] = parameters => { 
                return Response.AsJson(VegetableManager.GetAvailableVegetables()); 
            };
        }
    }
}

Note that despite the fact our service is "homed" in the `/services` path we still pass this URL part into the parent constructor - our ASP.Net application (the Umbraco install base) is still rooted at "/".

### Root Level Web.config
The following is a fairly detailed snapshot of the root-level Web.config file. Note that the Nancy handlers are placed immediately following the `remove` lines in the handler sections.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    :
    :  standard umbraco gubbins
    <appSettings>
        :
        <add key="umbracoReservedPaths" value="~/umbraco,~/install/,~/services" />
        :
    </appSettings>
    <system.web>
        :
        :
        <httpModules>
             <!-- URL REWRTIER -->
             <add name="UrlRewriteModule" type="UrlRewritingNet.Web.UrlRewriteModule, UrlRewritingNet.UrlRewriter" />
             <add name="umbracoRequestModule" type="umbraco.presentation.requestModule" />
             <!-- UMBRACO -->
             <add name="viewstateMoverModule" type="umbraco.presentation.viewstateMoverModule" />
             <add name="umbracoBaseRequestModule" type="umbraco.presentation.umbracobase.requestModule" />
             <!-- ASPNETAJAX -->
             <add name="ScriptModule" type="System.Web.Handlers.ScriptModule, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
             <!-- CLIENT DEPENDENCY -->
             <add name="ClientDependencyModule" type="ClientDependency.Core.Module.ClientDependencyModule, ClientDependency.Core" />
        </httpModules>
        <httpHandlers>
            <remove verb="*" path="*.asmx" />
            <add verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="services/*" />
            <!-- ASPNETAJAX -->
            <add verb="*" path="*.asmx" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" validate="false" />
            <add verb="*" path="*_AppService.axd" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" validate="false" />
            <add verb="GET,HEAD" path="ScriptResource.axd" type="System.Web.Handlers.ScriptResourceHandler, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" validate="false" />
            <!-- UMBRACO CHANNELS -->
            <add verb="*" path="umbraco/channels.aspx" type="umbraco.presentation.channels.api, umbraco" />
            <add verb="*" path="umbraco/channels/word.aspx" type="umbraco.presentation.channels.wordApi, umbraco" />
            <add verb="*" path="DependencyHandler.axd" type="ClientDependency.Core.CompositeFiles.CompositeDependencyHandler, ClientDependency.Core " />
            <add verb="GET,HEAD,POST" path="GoogleSpellChecker.ashx" type="umbraco.presentation.umbraco_client.tinymce3.plugins.spellchecker.GoogleSpellChecker,umbraco" />
        </httpHandlers>
        :
        :
    </system.web>
    :
    :
    <system.webServer>
        <modules runAllManagedModulesForAllRequests="true">
            <remove name="ScriptModule" />
            <remove name="UrlRewriteModule" />
            <remove name="umbracoRequestModule" />
            <remove name="viewstateMoverModule" />
            <remove name="umbracoBaseRequestModule" />
            <remove name="ClientDependencyModule" />
            <!-- Needed for login/membership to work on homepage (as per http://stackoverflow.com/questions/218057/httpcontext-current-session-is-null-when-routing-requests) -->
            <remove name="FormsAuthentication" />
            <add name="UrlRewriteModule" type="UrlRewritingNet.Web.UrlRewriteModule, UrlRewritingNet.UrlRewriter" />
            <add name="umbracoRequestModule" type="umbraco.presentation.requestModule" />
            <add name="viewstateMoverModule" type="umbraco.presentation.viewstateMoverModule" />
            <add name="umbracoBaseRequestModule" type="umbraco.presentation.umbracobase.requestModule" />
            <add name="ScriptModule" preCondition="managedHandler" type="System.Web.Handlers.ScriptModule, System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
            <add name="ClientDependencyModule" type="ClientDependency.Core.Module.ClientDependencyModule, ClientDependency.Core" />
            <!-- Needed for login/membership to work on homepage (as per http://stackoverflow.com/questions/218057/httpcontext-current-session-is-null-when-routing-requests) -->
            <add name="FormsAuthentication" type="System.Web.Security.FormsAuthenticationModule" />
        </modules>
        <handlers accessPolicy="Read, Write, Script, Execute">
            <remove name="WebServiceHandlerFactory-Integrated" />
            <remove name="ScriptHandlerFactory" />
            <remove name="ScriptHandlerFactoryAppServices" />
            <remove name="ScriptResource" />
            <remove name="Channels" />
            <remove name="Channels_Word" />
            <remove name="ClientDependency" />
            <remove name="SpellChecker" />
            <add name="Nancy" verb="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" path="services/*" />
            <add name="ScriptHandlerFactory" verb="*" path="*.asmx" preCondition="integratedMode" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
            <add name="ScriptHandlerFactoryAppServices" verb="*" path="*_AppService.axd" preCondition="integratedMode" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
            <add name="ScriptResource" verb="GET,HEAD" path="ScriptResource.axd" preCondition="integratedMode" type="System.Web.Handlers.ScriptResourceHandler, System.Web.Extensions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
            <add verb="*" name="Channels" preCondition="integratedMode" path="umbraco/channels.aspx" type="umbraco.presentation.channels.api, umbraco" />
            <add verb="*" name="Channels_Word" preCondition="integratedMode" path="umbraco/channels/word.aspx" type="umbraco.presentation.channels.wordApi, umbraco" />
            <add verb="*" name="ClientDependency" preCondition="integratedMode" path="DependencyHandler.axd" type="ClientDependency.Core.CompositeFiles.CompositeDependencyHandler, ClientDependency.Core " />
            <add verb="GET,HEAD,POST" preCondition="integratedMode" name="SpellChecker" path="GoogleSpellChecker.ashx" type="umbraco.presentation.umbraco_client.tinymce3.plugins.spellchecker.GoogleSpellChecker,umbraco" />
        </handlers>
        :
        :
        <validation validateIntegratedModeConfiguration="false" />
    </system.webServer>
    :
    :
</configuration>  
```
### Service Folder Web.config
Within the `/services` folder of our application another Web.config file exists that contains the following:
```xml
<?xml version="1.0"?>
<configuration>
  <system.web>
    <httpHandlers>
      <add path="*" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" verb="*"/>
    </httpHandlers>
  </system.web>
  <!-- Required for IIS 7.0 -->
  <system.webServer>
    <modules runAllManagedModulesForAllRequests="true"/>
    <validation validateIntegratedModeConfiguration="false" />
    <handlers>
      <add path="*" name="Nancy" type="Nancy.Hosting.Aspnet.NancyHttpRequestHandler" verb="*" preCondition="integratedMode" resourceType="Unspecified" allowPathInfo="true" />
    </handlers>
  </system.webServer>
</configuration>
```

We can now hit our simple service at `oursite.domain/services/Vegetables`
