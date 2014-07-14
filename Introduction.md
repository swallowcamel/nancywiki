First of all, welcome to Nancy! These guides will give you a quick tour of some of the features in Nancy without diving into too many details. There will be more to things than meets the eye so we invite you to explore Nancy on your own as well!

Nancy is a lightweight, low-ceremony, framework for building HTTP based services on .Net and [Mono](http://mono-project.com). The goal of the framework is to stay out of the way as much as possible and provide a super-duper-happy-path to all interactions.

This means that everything in Nancy is setup to have sensible defaults and conventions, instead of making you jump through hoops and go through configuration hell just to get up and running. With Nancy you can go from zero to website in a matter of minutes. Literally.

Nancy is designed to handle `DELETE`, `GET`, `HEAD`, `OPTIONS`, `POST`, `PUT` and `PATCH` requests and provides a simple, elegant, [Domain Specific Language (DSL)](http://en.wikipedia.org/wiki/Domain-specific_language) for returning a response with just a couple of keystrokes, leaving you with more time to focus on the important bits.. **your** code and **your** application.

All of this is built, by the community, as an open-source framework, meaning you get full access to the source code, and is licensed under the [MIT license](http://www.opensource.org/licenses/mit-license.php).

You can get Nancy from [Nuget](http://nuget.org), [our TeamCity server](http://teamcity.codebetter.com/project.html?projectId=project112&tab=projectOverview&guest=true) (for latest builds) or download the source from our [GitHub repositories](http://nancyfx.org).

## Built to run anywhere

Nancy is built to run anywhere and we mean it. Right from the start, Nancy was designed to not have any dependencies on existing frameworks. Built with the [.NET framework client profile](http://msdn.microsoft.com/en-us/library/cc656912.aspx), Nancy can be used pretty much wherever you want to, since it’s completely self contained with its own request and response objects.

One of the core concepts in Nancy is hosts. A host acts as an adaptor for a hosting environment and Nancy, thus enabling Nancy to run on existing technologies such as ASP.NET, WCF and [OWIN](http://owin.org), or integrated in any given application.

Specific host implementations are not shipped with the core Nancy framework. They are shipped separately, as are many other additional functionalities such as forms authentication, from the sources mentioned earlier. Building a Nancy application is like picking your favourite parts from a web framework buffet! Usually the bare minimum you will use when building a Nancy service are the core framework and a host.

## The super-duper-happy-path

The “super-duper-happy-path” (or **SDHP** if you’re ‘down with the kids’ :wink:) is a phrase we coined to describe the ethos of Nancy; and providing the “super-duper-happy-path” experience is something we strive for in all of our APIs.

While it’s hard to pin down exactly what it is, it’s a very emotive term after all, but the basic ideas behind it are:

* “It just works” - you should be able to pick things up and use them without any mucking about. Added a new module? That’s automatically discovered for you. Brought in a new View Engine? All wired up and ready to go without you having to do anything else. Even if you add a new dependency to your module, by default we’ll locate that and inject it for you - no configuration required.
* “Easily customisable” - even though “it just works”, there shouldn’t be any barriers that get in the way of customisation should you want to work the way you want to work with the components that you want to use. Want to use another container? No problem! Want to tweak the way routes are selected? Go ahead! Through our bootstrapper approach all of these things should be a piece of cake.
* “Low ceremony” - the amount of “Nancy code” you should need in your application should be minimal. The important part of any Nancy application is your code - our code should get out of your way and let you get on with building awesome applications. As a testament to this it’s actually possible to fit a functional Nancy application into a single Tweet :-) 
* “Low friction” - when building software with Nancy the APIs should help you get where you want to go, rather than getting in your way. Naming should be obvious, required configuration should be minimal, but power and extensibility should still be there when you need it.

Above all, creating an application with Nancy should be a pleasure, and hopefully fun! But without sacrificing the power or extensibility that you may need as your application grows.

## Creating your first Nancy application

Enough talk, let’s see some code! We are going to assume that you have [Nuget](http://nuget.org) installed and are using Visual Studio 2010, however, this works equally as well on [Mono](http://mono-project.com) (using version 2.10.2 or later) and [MonoDevelop](http://monodevelop.com). We’re going to build the ubiquitous “hello world” application using Nancy and Nancy’s ASP.NET hosting.

1. Install the [Nancy project templates](http://visualstudiogallery.msdn.microsoft.com/f1e29f61-4dff-4b1e-a14b-6bd0d307611a).
1. Create a new `Nancy Empty Web Application with ASP.NET Hosting`
1. Add a `Nancy module`, which is a standard C# class, and define a route handler for the root URL of the web application, by adding a small amount of code to the constructor:
1. Compile and run to see the result!

The HelloModule.cs code

```c#
public class HelloModule : NancyModule
{
    public HelloModule()
    {
        Get["/"] = parameters => "Hello World";
    }
}
```

##More Info

* [[Why use NancyFX?|http://blog.jonathanchannon.com/2012/12/19/why-use-nancyfx/]]

[Documentation overview](Documentation) - [Part 2. Exploring the Nancy module >>](Exploring the Nancy module)