_Please do not edit this list directly. Features should be discussed at [the Nancy User Group](https://groups.google.com/forum/?pli=1#!topic/nancy-web-framework/-u73fTImRCk) before they are selected for the release candidate target. This does not mean that contributions for features that are not on the list will be rejected - they are, as always, welcome at all times!_

## View Engines 
* <del>Caching support for caching intermediary representations, rather than rebuilding the view from the template each time. _**(TheCodeJunkie, RobertTheGrey, Grumpydev)**_</del>
* <del>Partial view support - potentially need to inject an "IViewHelper" object that allows access to render a partial view and utility functions like html encode. _**(TheCodeJunkie, RobertTheGrey, Grumpydev)**_</del>
* Check to see which view engines play nicely in client profile.
* <del>Support conventions for looking up views, layouts and so on **_(TheCodeJunkie)_**</del>

## Bootstrappers
* <del>Add a default ASP.Net bootstrapper using the HttpContext for per-request lifetime. **_(Grumpydev)_**</del>
* Check to see which containers play nicely in client profile.

## Hosting
* Make sure the context is disposed correctly in WCF and standalone hosts.
* <del>Stop the ASP.Net hosting spinning up multiple bootstrappers in some scenarios (otherwise known "doing a @lucisferre" ;-))</del>
* <del>Introduce RequestStream in the Nancy hosts and engine **_(TheCodeJunkie)_**</del>

## OOTB Pipeline Helpers
* <del>Conventions based authentication/claims based module hooks (similar to the security demo) **_(Grumpydev)_**</del>
* <del>Forms based authentication **_(Grumpydev)_**</del>
* Static file app level pre request hook (should support embedded resources!) **_(Grumpydev)_**
* Automatic conditional GET support app level post request hook (ETag and Modified) **_(Grumpydev)_**
* Custom error pages post request hook

## Diagnostics
* List discovered modules **_(TheCodeJunkie)_**
* List routes in a module **_(TheCodeJunkie)_**
* Query a route and see which routes that are matched using the configured IRoutePatternMatcher **_(TheCodeJunkie)_**
* View default registrations **_(TheCodeJunkie)_**
* .. add lots more stuff **_(TheCodeJunkie)_**

## Mono
* Work on official support for Nancy on Mono 2.10
* Make sure xunit, fakeiteasy and mspec runs on Mono 2.10
* Setup CI environment for mono builds

## Documentation
* Make sure all public types and members are documented
* Write documentation for all existing features
* Write documentation on how to extend Nancy
* Bring wiki up to date

## Testing
* Move ShouldExtensions from Nancy.Tests to Nancy.Testing
* <del>Look into simplifying unit testing of Nancy modules **_(TheCodeJunkie)_**</del>
* <del>Possibly create a test harness for Nancy applications that you can use in your tests **_(TheCodeJunkie)_**</del>

## Repositories
* <del>Create origanisation</del>
* Split out bootstrappers for third party containers

## Misc
* Cleanup the cookies so request and response have the same structure.
* Migrate session to DynamicDictionary (while maintaining current "is dirty" features).
* <del>Make NuGet packages for the core, hosting options etc.</del>
* Add ViewData support (*)
* Make sure performance and memory profiles are ok
* <del>Add support for multipart/form-data **_(TheCodeJunkie)_**</del>
* Add support for header continuations in multipart/form-data encoded data **_(TheCodeJunkie)_**
* Add support for encoded filenames in multipart/form-data encoded data **_(TheCodeJunkie)_**
* <del>Support large file uploads by storing stream on disk and swap out Request.Body **_(TheCodeJunkie)_**</del>
* Set content-length on responses (?)
* <del>Add Nancy version response header (Nancy-Version?)</del>
* <del>Create "Empty Nancy Web Application" project template (no binaries included, default view engine template conventions and so on)</del>
* Automate NuGet generation and publishing
* Add request.body helpers, for formats such as json and xml (Request.Body.Json())
* <del>Add model-binding **_(Grumpydev)_**</del>

_* this may be a joke ;-)_