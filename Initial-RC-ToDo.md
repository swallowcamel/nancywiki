## View Engines
* Caching support for caching intermediary representations, rather than rebuilding the view from the template each time.
* Partial view support - potentially need to inject an "IViewHelper" object that allows access to render a partial view and utility functions like html encode.
* Check to see which view engines play nicely in client profile.

## Bootstrappers
* Add a default ASP.Net bootstrapper using the HttpContext for per-request lifetime.
* Check to see which containers play nicely in client profile.

## Hosting
* Make sure the context is disposed correctly in WCF and standalone hosts.
* Stop the ASP.Net hosting spinning up multiple bootstrappers in some scenarios (otherwise known "doing a @lucisferre" ;-))

## OOTB Pipeline Helpers
* Conventions based authentication/claims based module hooks (similar to the security demo)
* Static file app level pre request hook
* Automatic conditional GET support app level post request hook (ETag and Modified)
* Custom error pages post request hook

## Diagnostics
* List discovered modules
* List routes in a module
* Query a route and see which routes that are matched using the configured IRoutePatternMatcher
* .. add lots more stuff

## Mono
* Work on official support for Nancy on Mono 2.10
* Setup CI environment for mono builds

## Documentation
* Make sure all public types and members are documented
* Write documentation for all existing features
* Write documentation on how to extend Nancy

## Misc
* Cleanup the cookies so request and response have the same structure.
* Migrate session to DynamicDictionary (while maintaining current "is dirty" features).
* Remove bootstrapper projects and move them into the "accessories" project.
* Make NuGet packages for the core, hosting options etc.
* Add ViewData support (*)
* Make sure performance and memory profiles are ok
* Add support for multipart/form-data

_* this may be a joke ;-)_