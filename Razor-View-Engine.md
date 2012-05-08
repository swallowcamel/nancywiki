The Razor engine in Nancy is a custom implementation built around the Razor syntax parser. Please note that the implementation may have differences to implementation used by ASP.NET MVC.

## Installing Razor

Simply reference `Nancy.ViewEngines.Razor.dll` (preferbly by installing the `Nancy.ViewEngines.Razor` nuget) and return views ending in `cshtml` or `vbhtml` it's that simple.

## Configuring Razor ##

You can specify assemblies and default namespaces that Razor needs to use whilst compiling the views by bootstrapping your own `IRazorConfiguration` implementationm thus removing the need to add the `@using` statements to each view. _This step is totally optional_ if you don't require additional references or namespaces in your view.

The default `IRazorConfiguration` implementation (automatically used by Nancy unless explicitly overridden in the bootstrapper) looks in `app\web.config` in the razor section.

Step 1: Create a custom configuration section

```xml
<configSections>
	<section name="razor" type="Nancy.ViewEngines.Razor.RazorConfigurationSection, Nancy.ViewEngines.Razor" />
</configSections>
```
Step 2: Configure Razor! _(note! this is just a sample configuration)_

```xml
<razor disableAutoIncludeModelNamespace="false">
	<assemblies>
		<add assembly="System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
		<add assembly="SolrNet" />
		<add assembly="SyslogServerLibrary" />
	</assemblies>
	<namespaces>
		<add namespace="SolrNet" />
		<add namespace="SyslogServerLibrary" />
	</namespaces>
</razor>
```
Pretty self explanatory except `disableAutoIncludeModelNamespace` which by default auto references the assembly of the model you pass into the view.

## Letting Razor know what base type your views use
You can let Razor know which base type your views are using, thus gaining intellisense access to the members of the base type at design-time, by using the `@inherits` statement in your views. For example you could specify `@inherits Nancy.ViewEngines.Razor.NancyRazorViewBase<dynamic>` to use the `NancyRazorViewBase` base type with a `dynamic` model. You need to do this in each of the views where you want the design-time candy.

However there is another way if you have ASP.NET MVC installed. Visual Studio has an intellisense sub-system that can be used to teach Visual Studio about different syntaxes. This sub-system is a bit cumbersome to beat into submission and using it would require us to provide an install for a Nancy toolkit. 

ASP.NET MVC has built its own abstraction on top of this sub-system, which is installed as a visual studio extension when you run the ASP.NET MVC installer (it's installed in a different folder than the default extensions folder, thus is not shown in the extension manager. Sneaky). If you have ASP.NET MVC installed on the same machine as you are doing Nancy development, you can tap into this abstraction and work for you and your Nancy application. 

To do this you need to add the following to your `app\web.config` file

```xml
<configSections>
    <sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
        <section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
        <section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
    </sectionGroup>
</configSections>
<system.web.webPages.razor>
    <pages pageBaseType="Nancy.ViewEngines.Razor.NancyRazorViewBase`1[[System.Object]]">
        <namespaces>
            <add namespace="Nancy.ViewEngines.Razor" />
        </namespaces>
    </pages>
</system.web.webPages.razor>
```

Not that the `Nancy.ViewEngines.Razor.NancyRazorViewBase`1[[System.Object]]` part is a generic type name of `NancyRazorViewBase<object>` and you can swap out the generic type for what ever type your model is. It's also worth noting that the `Version` and `PublicKeyToken`, of the ASP.NET MVC assembly references, will be different depending on which version of ASP.NET MVC you have installed (these are for ASP.NET MVC 3).