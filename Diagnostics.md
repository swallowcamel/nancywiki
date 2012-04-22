Nancy has a built in diagnostics dashboard that enabled you to inspect the state and behavior of your application at runtime. The dashboard can easily be extended with custom functionality and can be tailored to suite the needs of your application.

The diagnostics dashboard is embedded in the main Nancy assembly. The back-end is a Nancy application that is serving a backbone.js front-end. It also makes use of the `SuperSimpleViewEngine` to render out views. It is built to be fast, extendible and cross-browser compliant.

You reach the dashboard by pointing your browser to `http://<address-of-your-application>/_Nancy/`. However before being able to use the dashboard, you first need to configure it.

## Configuring access to the dashboard

The dashboard is automatically available to you, in you application, however in order to gain access you need to configure a password. Without the password you will be presented with a page telling you that the dashboard needs to be configured inorder for you to use it, along with instructions on how to perform the configuration.

To configure a password for your diagnostics dashboard, you need to override the `DiagnosticsConfiguration` property of your [[Bootstrapper]]. Once overriden, you should return an instance of the `DiagnosticsConfiguration` that has had a password assigned to the `Password` property.

    public class CustomBootstrapper : DefaultNancyBootstrapper
    {
        protected override DiagnosticsConfiguration DiagnosticsConfiguration
        {
            get { return new DiagnosticsConfiguration { Password = @"A2\6mVtH/XRT\p,B"}; }
        }
    }

When this has been done, you will be presented with a login screen instead of a configuration notification, the next time your browse the dashboard. A valid login will produce a login cookie that has a sliding expiration set to 15 minutes. 

Each access to the dashboard will extend the lifetime, of the cookie, for another 15 minutes. If the cookie expires, before you revisit the dashboard, you will be redirected back to the login form.

## Removing diagnostics

As mentioned earlier, the diagnostics dashboard is automatically wired up for your application. However, should you wish to completley remove the dashboard you can do so by performing a call to `DiagnosticsHook.Disable()` and it will no longer be accessible.

It is recommended that the call to `DiagnosticsHook.Disable()` is performed from the `ApplicationStartup` method in your bootstrapper. This will ensure that the dashboard is disabled as soon as your application starts up.

    public class CustomBootstrapper : DefaultNancyBootstrapper
    {
        protected override void ApplicationStartup(TinyIoC.TinyIoCContainer container, IPipelines pipelines)
        {
            DiagnosticsHook.Disable();
        }
    }

## So what kind of tools are there?

The dashboard presents you with four options when you access it; `Information`, `Interactive Diagnostics`, `Request Tracing` and `Configuration`. These are the main section of the dashboard and your gateway to taking a peek under the hood of your application at runtime.

### Information

The information page provides information about the setup of the Nancy environment, such as assembly version, which components that are being used by Nancy, what view engines that are availalble, what host is being used and so on.

### Configuration

In Nancy the `StaticConfiguration` type can be used to control the behavior of your application, for example controlling the case-sensitivity, if caching should be used and so on.

The configuration page provides access to the `StaticConfiguration` though a user interface, so that the behavior of the application can be altered at runtime.

It should be noted that any changes that are applied though the configuration page are not persisted and will be reset, to their configured value, when the application is restarted.

To configure these behaviors permanently you should still alter their value by setting them from the `ApplicationStartup` method in your bootstrapper.

### Request tracing

As you would expect, request tracing, provided a window for inspecting the behavior of individual requests as they are processed by Nancy.

For performance reasons, request tracing is diasabled by default and can either be turned on using the `Configuration` page on the dashboard or by setting the `StaticConfiguration.EnableRequestTracing` property to `true` 

    public class CustomBootstrapper : DefaultNancyBootstrapper
    {
        protected override void ApplicationStartup(TinyIoC.TinyIoCContainer container, IPipelines pipelines)
        {
            StaticConfiguration.EnableRequestTracing = true;
        }
    }

The trace log is accessible though the `NancyContext` and it is easy to add your own information, in order to provider an even richer experience for your application.

    public class HomeModule : NancyModule
    {
        public HomeModule()
        {
            Get[“/“] = parameters => {
                this.Context.Trace.TraceLog.WriteLog(s => s.AppendLine(“Root path was called”));

                return HttpStatusCode.Ok;
            };
        }
    }

The `WriteLog` method takes a function that will be passed an instance of a `StringBuilder` at runtime. The reason it is take a function is that when diagnostics is disabled, Nancy will simply not invoke the function so there will be no performance penalties for leaving the trace code in your application. 

### Interactive diagnostics

What if you could query your application about pretty much anything during runtime? This is exactly what the interactive diagnostics feature enables you to do. It does not provide a query language for you to write expressions in (even though it would be possible for you to add in your own application if you so wish), but it enables you to implement diagnostics tools as normal C# types and then invoke them from the dashboard.

This enables you to use the full capabilities of the language, platform and Nancy (including dependency injection) to deliver pretty much any diagnostics capability you can think of. As long as your type implements the `IDiagnosticsProvider` interface, then the Nancy diagnostics will automatically discover it and expose it as an interactive diagnostics tool.

#### The IDiagnosticsProvider interface

Anything that implements this type lets Nancy know that it is capable of providing interactive diagnostics tools for the dashboard. The interface it self is very basic and provides mostly metadata about the provider

    /// <summary>
    /// Defines the functionality a diagnostics provider.
    /// </summary>
    public interface IDiagnosticsProvider
    {
        /// <summary>
        /// Gets the name of the provider.
        /// </summary>
        /// <value>A <see cref="string"/> containing the name of the provider.</value>
        string Name { get; }

        /// <summary>
        /// Gets the description of the provider.
        /// </summary>
        /// <value>A <see cref="string"/> containing the description of the provider.</value>
        string Description { get; }

        /// <summary>
        /// Gets the object that contains the interactive diagnostics methods.
        /// </summary>
        /// <value>An instance of the interactive diagnostics object.</value>
        object DiagnosticObject { get; }
    }

The most important member, of the `IDiagnosticsProvider` interface, is the `DiagnosticObject` property. The object returned by this property is the one that will be wired up on the dashboard. 

#### So what does the diagnostics object look like?

There is nothing special about a diagnostics object. It does not have to be the same type as the one that implement the `IDiagnosticsProvider` interface, nor does it have to implement any other interfaces. The thing to understand is that any public method on the object are going to be exposed on the iteractive diagnostics dashboard.

These methods can have primitive (string, int and so on) parameters and return any object, as long as it is possible to JSON serialize it. This means that method invocation will be strongly-typed and no need to perform cast or type checks.

When you want to invoke one of these methods, from the diagnostics dashboard, Nancy present you with a form that contains input fields that will be used to provide values to the methods parameter once it is invoked. The input fields contains information about the parameter name and type.

The output of the method will then be rendered using the [JSON Report Format](http://www.servicestack.net/docs/framework/json-report-format) script, created by [ServiceStack](http://servicestack.net).

#### Providing a description of your method

It is possible to provide a description for a diagnostics method that will be included in the form that is used to invoke the method. 

There are two ways you can provide the description; by using an attribute on the method or by providing a property that will return the desciption. 

To use the attribute you simply add the `Nancy.Diagnostics.DescriptionAttribute` to the method and provide a description and that is all there is to it. However sometimes you may find yourself requiring a bit more control of the description string, for example of you want to provide localized strings.

When you find yourself in these situations you can use the property approach. Nancy use a convention where it tries to find a property that as the same name as the method, but with an added `Description` suffix appended to it. An example of this would be a `NameOfYourMethodDescription` property for the `NameOfYourMethod` method.

If your method has been decorated with the `DescriptionAttribute` and you also have a description property declared, then the propery will take presedence over the attribute.

#### Customizing the output with templates

As previously mentioned, the output of your diagnostics method will be rendered using the [JSON Report Format](http://www.servicestack.net/docs/framework/json-report-format) and will use the formatting templates that it declares. However, Nancy provides you with the option use custom output templates.

An output template is a `Mustache` template and you can specify a unique template for each of your diagnostics method. Just like with method descriptions, you have the possibility, to provide the template, using either an attribute or a property.

For templates the attribute that is used is the `Nancy.Diagnostics.TemplateAttribute` and the naming convention for the property is name of the property with a `Template` suffix.

#### Creating a simple diagnostics provider

Let's create a simple diagnostics provider with a method that will greet someone by their name. Not really a real scenario but it is a basic enough sample to show the anatomy of a diagnostics provider without getting distored by the implementation.

    public class CustomDiagnosticsProvider : IDiagnosticsProvider
    {
        public string Name
        {
            get { return "Custom diagnostics provider"; }
        }

        public string Description
        {
            get { return "Provides custom diagnostics capabilities"; }
        }

        public object DiagnosticObject
        {
            get { return this; }
        }

        [Description("Greets a person using their name")]
        [Template("<h1>{{model.Result}}</h1>")]
		public string Greet(string name)
        {
            return string.Concat("Hi, ", name);
        }
    }

The provided exposes a single method, the `Greet`method. The method takes a single string parameter which is the name of the person to greet. In addition the method has been given a description, using the `DescriptionAttribute`, that will be displayed in the form that will be used to invoke the method from the dashboard.

Instead of using the default output format, the method also provides a custom `Mustache` template that will be used to format the output.

A proper diagnostics provider would probably take a couple of contstructor dependencies in order to be able to access the various pieces of the application.


[<< Part 12. Managing static content](Managing static content) - [Documentation overview](Documentation)