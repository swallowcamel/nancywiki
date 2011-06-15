**Installing Razor**
Simply reference `Nancy.ViewEngines.Razor.dll` and return views ending in `cshtml` or `vbhtml` it's that simple. Please note that as of yet certain feautures you might be used to from MVC/ASP.NET are unavailable such as strongly typed views, layouts and renderpartial. You can sorta mock strongly typing views, for those of us who have complex object graphs, by specifing 

	@MyType model = (MyType)Model;

at the top of your view. 

**Configuring Razor.**

You can specify assemblies and default namespaces that razor needs to use whilst compiling the views by bootstrapping your own IRazorConfiguration implementation. 

The default RazorConfiguration looks in app or web.config in the razor section.

Step 1: Create a custom configSection

	<configSections>
		<section name="razor" type="Nancy.ViewEngines.Razor.RazorConfigurationSection, Nancy.ViewEngines.Razor" />
	</configSections>

Step 2: Configure Razor!

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

Pretty self explanatory except `disableAutoIncludeModelNamespace` which by default auto references the assembly of the model you pass into the view.

Step 3: Use Razor!

Inside NancyModule Constructor
     `...`
`     Get["/Home/Index"] = param =>`
`                          {`
`                              var model = repo.GetSomeModel();`
`                              return View["/MyViewPath/Index.cshtml",model]`
`                          };`
 
Note that the path for the razor view is relative to the root of your project.


