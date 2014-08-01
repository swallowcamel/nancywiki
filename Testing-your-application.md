The super-duper-happy-path (see [[Introduction]]) is not just something that we want to be the ethos for writing Nancy applications, but also for testing them. For that reason we provide a special assembly called `Nancy.Testing`, available via [Nuget](http://nuget.org), [our TeamCity server](http://teamcity.codebetter.com/project.html?projectId=project112&tab=projectOverview&guest=true) (for latest builds) or from the [GitHub repository](https://github.com/NancyFx/Nancy/tree/master/src/Nancy.Testing), and it contains some nice helpers.

_Note!_ You should always keep your tests in a separate assembly than that of your application. The reason for this is the auto-discoverability that is used by Nancy to find and wire up a lot of things, including the [[Bootstrapper]]. If you keep your tests in the same assembly then isolation is going to be a lot more difficult. 

Because Nancy is a self-composing framework (see [[Bootstrapper]]), the runtime composition of the framework can have impact on the results of your routes. To help you test your routes in the right runtime context, we provide a helper class called `Browser`.

You instantiate the Browser with the bootstrapper you want to use when the request is processed, and then fire off requests and inspect the results.
```c#
[Fact]
public void Should_return_status_ok_when_route_exists()
{
    // Given
    var bootstrapper = new DefaultNancyBootstrapper();
    var browser = new Browser(bootstrapper);
     
    // When
    var result = browser.Get("/", with => {
        with.HttpRequest();
    });
        
    // Then
    Assert.Equal(HttpStatusCode.OK, result.StatusCode);
}
```
The test will send a `GET` request to your application, running in the configuration provided by the `DefaultNancyBootstrapper` and return the actual result, wrapped in an instance of the `BrowserResult` type.

You can send all supported request types with the Browser class, and the configuration lets you specify things as request protocol, headers, form values, uploaded files and much more. The configuration can easily be extended with the use of extension methods.

As an alternative to specifying the full configuration when sending the request you can set up configuration defaults when you create the Browser object. Everything you can set up when sending the request, you can set up as defaults. 

For instance this will send a `GET` request over `HTTP` with an accept header set to `application/json`.
```c#
[Fact]
public void Should_return_status_ok_when_route_exists()
{
    // Given
    var bootstrapper = new DefaultNancyBootstrapper();
    var browser = new Browser(bootstrapper, defaults: to => to.Accept("application/json"));
     
    // When
    var result = browser.Get("/", with => {
        with.HttpRequest();
    });
        
    // Then
    Assert.Equal(HttpStatusCode.OK, result.StatusCode);
}
```

The `BrowserResult` type will give you full access to the `NancyContext`, that was used during the processing of the request and also give you access to the actual request and response objects as well.

## Controlling the execution environment

Sometimes you are going to want more fine grained control over the composition of Nancy that is being used to execute the test. In these situations, using the `DefaultNancyBootstrapper`, or sub class of it, won’t give you the control you are going to need.

For this reason, the Nancy.Testing assembly contains a bootstrapper called `ConfigurableBootstrapper` that provides an API for overriding anything in Nancy with another type or an actual instance.

Being able to tell the bootstrapper to use a specific instance is very powerful, it would, for example, enable you to use mocks in the composition of the framework and do assertions on those in your tests.

## Being assertive

Once you’ve executed your request and retrieved your response you’re going to want to run some assertions to make sure everything is a-ok. The Nancy testing library provides a number of helpers to make asserting common things easy and, more importantly, to keep the intent of your tests clear and concise.

For example - if you were testing a login page and wanted to verify that the user was redirected to an “error” page if they entered incorrect credentials you could use the `ShouldHaveRedirectedTo` assertion:
```c#
[Fact]
public void Should_redirect_to_login_with_error_details_incorrect()
{
    // Given
    var bootstrapper = new DefaultNancyBootstrapper();
    var browser = new Browser(bootstrapper);
        
    // When
    var response = browser.Post("/login/", (with) => {
        with.HttpRequest();
        with.FormValue("Username", "username");
        with.FormValue("Password", "wrongpassword");
    });
        
    // Then
    response.ShouldHaveRedirectedTo("/login?error=true&username=username");
}
```
In addition to general asserts about the response, Nancy also provides the ability to assert against the content of returned views by picking out elements of the page using CSS3 selectors, and asserting various facts about them. 

As an example, following on from the test fixture above, you may want to assert that the “error” page that the user has been redirected to actually displays an error message - after all, it’s no good redirecting to an error page if there’s no error!

To do this what we want to do is make sure a certain element with an id of “errorBox” is on the page, is set to a certain style, and contains a certain snippet of text. Sounds quite complicated, but it’s a piece of cake with the helpers:
```c#
[Fact]
public void Should_display_error_message_when_error_passed()
{
    // Given
    var bootstrapper = new DefaultNancyBootstrapper();
    var browser = new Browser(bootstrapper);
     
    // When
    var response = browser.Get("/login", (with) => {
        with.HttpRequest();
        with.Query("error", "true");
    });
     
    // Then
    response.Body["#errorBox"]
            .ShouldExistOnce()
            .And.ShouldBeOfClass("floatingError")
            .And.ShouldContain(
                "invalid",
                StringComparison.InvariantCultureIgnoreCase);
}
```
In this test we use the browser object to create a request that’s equal to the url we are sent to in error (the same one we verified earlier), then verify the existence and properties of the errorBox element.

This particular sample shows a very simple selector, but you can use the full power of CSS3 selectors ([http://www.w3.org/TR/css3-selectors/](http://www.w3.org/TR/css3-selectors/)) to grab elements from the HTML and run asserts against them.

## Getting more out of your result

There's a number of extension methods that can prove very handy for the aspiring test-driven-developer. For example, the `BrowserResult` object mentioned above is awesome, because it shows everything that is rendered on the browsers. But it doesn't show what created the `BrowserResult`. Luckily the `GetModel<T>()` extension is there to help you. 

Here's a very contrived example module and model class, from the unit tests, that we will use to show these extension methods:

```c#
public class AModuleToTestExtensionMethodsWith : NancyModule
{
  private const string VIEW_PATH = "TestingViewExtensions/ViewFactoryTest.sshtml";

  public AModuleToTestExtensionMethodsWith() : base("/a/long/path")
  {
     this.Get["/testingViewFactory"] = _ => 
           this.View[VIEW_PATH, new ViewFactoryTestModel{ AString = "A value" }];
  }
}

public class ViewFactoryTestModel
{
  public string AString { get; set; }
}
```

You can now get hold of the model and it's value in your unit tests by using the ```GetModel<T>()``` extension method. For example like this:

```c#
public class GetModelExtententionsTests
{
    private readonly Browser _browser;
    public GetModelExtententionsTests()
    {
        this._browser = new Browser(with => {
           with.Module<AModuleToTestExtensionMethodsWith>();
           with.ViewFactory<TestingViewFactory>();
        });
    }

    [Fact]
    public void Can_get_the_model_and_read_the_values()
    {
       var response = this._browser.Get("/testingViewFactory");
       var model = response.GetModel<ViewFactoryTestModel>();
       Assert.Equal("A value", model.AString);
    }
}
```

**Note**: Use of these extension methods requires the use of the `TestingViewFactory`, set on the testing `Browser` object. It's a wrapper `ViewFactory` that saves the Model and then exposes it with the extension method. You don't have to think too hard about that though, only make sure to set the `TestingViewFactory` (e.g. `with.ViewFactory<TestingViewFactory>();`).

There are a couple of other extension methods as well, that can help you in testing your application:

* `GetViewName()` returns the name of the view that is rendered, with extension. For the example we showed above, this would return `"TestingViewExtensions/ViewFactoryTest.sshtml"`.
* `GetModuleName()` returns the name of the module that returned the response. In the example above, this call would return `"AModuleToTestExtensionMethodsWith"`.
* `GetModulePath()` returns the routing path to the Module. For example: `"/a/long/path/testingViewFactory"` would be returned, since our module uses the base class constructor with those parameters.

These methods could be helpful while testing the inner workings of your applications, which are not always visible in generated responses.

##  More Info

* [[Nancy and VB.Net: testing your modules.|http://blogs.lessthandot.com/index.php/WebDev/ServerProgramming/nancy-and-vb-net-testing]]
* [[Intro to testing with Nancy|http://www.marcusoft.net/2013/01/NancyTesting1.html]]
* [[The Configurable bootstrapper|http://www.marcusoft.net/2013/01/NancyTesting2.html]]
* [[The Browser and Response objects|http://www.marcusoft.net/2013/01/NancyTesting3.html]]
* [[Testing (razor) views|http://www.marcusoft.net/2013/02/NancyViewTesting.html]]
* [[Hat and shoeless testing with Simple.Data|http://www.marcusoft.net/2013/02/NancyTesting4.html]]
* [[Executable Specifications Through The Full Stack, In Memory|http://www.marcusoft.net/2013/02/NancyTesting5.html]]

[<< Part 12. Localization](Localization) - [Documentation overview](Documentation) - [Part 14. The root path >>](The root path)