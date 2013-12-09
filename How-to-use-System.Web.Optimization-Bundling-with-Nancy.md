1. Install the `Microsoft.AspNet.Web.Optimization` Nuget Package
2. Setup your bundles in the `ApplicationStartup` Method of your Bootstraper!
```
protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines) 
{
    BundleTable.Bundles.Add(new ScriptBundle().Add(....));
}
```
3. Add your respective folders to the `StaticContentsConventions` in the `ConfigureConventions` Method of the Bootstraper
```
protected override void ConfigureConventions(NancyConventions nancyConventions)
{
    nancyConventions.StaticContentsConventions.Add(StaticContentConventionBuilder.AddDirectory("/Scripts"));
}
```
**VERY IMPORTANT**
Don't forget to add the Urls you provided for the bundles here!
if you used `bundles.Add(new ScriptBundle("~/js"))`, you have to add `/js` to the  `StaticContentConventions`
too!
4. Add `System.Web` and `System.Web.Optimization` to your Razor Assemblies:
```
<razor disableAutoIncludeModelNamespace="false">
    <assemblies>
        <add assembly="System.Web.Optimization" />
        <add assembly="System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
    </assemblies>
</razor>
```
5. Don't use `@Scripts.Render()` but use `@Html.Raw(Scripts.Render())`