The super-duper-happy-path (see [[Introduction]]) is not just something that we want to be the ethos for writing Nancy applications, but also for testing them. For that reason we provide a special assembly called `Nancy.Testing`, available via [Nuget](http://nuget.org), [our TeamCity server](http://teamcity.codebetter.com/project.html?projectId=project112&tab=projectOverview&guest=true) (for latest builds) or from the [GitHub repository](https://github.com/NancyFx/Nancy/tree/master/src/Nancy.Testing), and it contains some nice helpers.

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
    Assert.Equal(result.StatusCode, HttpStatusCode.OK);
}
```
The test will send a `GET` request to your application, running in the configuration provided by the `DefaultNancyBootstrapper` and return the actual result, wrapped in an instance of the `BrowserResult` type.

You can send all supported request types with the Browser class, and the configuration lets you specify things as request protocol, headers, form values, uploaded files and much more. The configuration can easily be extended with the use of extension methods.

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

[<< Part 9. View location conventions](View location conventions) - [Documentation overview](Documentation) - [Part 11. The root path >>](The root path)