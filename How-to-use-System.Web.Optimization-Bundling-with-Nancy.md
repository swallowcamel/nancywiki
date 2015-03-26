**Note:** Microsoft.AspNet.Web.Optimization requires HttpContext, which is not available with self hosting.
 
1. **IF** you are using Nancy with **OWIN**, make sure you use
```
app.UseNancy(options => options.PassThroughWhenStatusCodesAre(HttpStatusCode.NotFound));
```
2. Install the `Microsoft.AspNet.Web.Optimization` Nuget Package
3. Setup your bundles in the `ApplicationStartup` Method of your Bootstrapper!
```
protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines) 
{
    BundleTable.Bundles.Add(new ScriptBundle().Add(....));
}
```
4. Add your respective folders to the `StaticContentsConventions` in the `ConfigureConventions` Method of the Bootstrapper
```
protected override void ConfigureConventions(NancyConventions nancyConventions)
{
    nancyConventions.StaticContentsConventions
        .Add(StaticContentConventionBuilder.AddDirectory("/Scripts"));
}
```
**VERY IMPORTANT**
Don't forget to add the Urls you provided for the bundles here!
if you used `bundles.Add(new ScriptBundle("~/js/mycombinedjsfiles").AddDirectory("~/Scripts/myScripts"))`, you have to add `/js` to the  `StaticContentConventions`
too, else the Optimized versions will not work once you go into Release mode!

5. Add `System.Web` and `System.Web.Optimization` to your Razor Assemblies:
```xml
<razor disableAutoIncludeModelNamespace="false">
    <assemblies>
        <add assembly="System.Web.Optimization" />
        <add assembly="System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
    </assemblies>
</razor>
```

# Example application

https://github.com/stanislavromanov/NancyAspNETWebOptimizationExample