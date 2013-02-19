We try to keep the number of breaking changes down to an absolute minimum, but sometimes we feel the change is so important that we decide to perform it. Many time these changes will not directly have an effect on users that upgrade their Nancy version, but since they are changes to public types and behaviors, we still list them here in case you should run into issues after an update.

## 0.17.0 ([View](https://github.com/NancyFx/Nancy/issues?milestone=22&state=closed))
- [First class static content] (https://github.com/NancyFx/Nancy/pull/982)
- [New route resolution engine] (https://github.com/NancyFx/Nancy/pull/988)
- [Disable CSRF hook by default] (https://github.com/NancyFx/Nancy/pull/989)
- [Scanning changes] (https://github.com/NancyFx/Nancy/pull/953)

## 0.16.1 ([View](https://github.com/NancyFx/Nancy/issues?milestone=23&state=closed))
- None

## 0.16 ([View](https://github.com/NancyFx/Nancy/issues&milestone=16&state=closed))
- [Changed RootPathProvider to return an instance](https://github.com/NancyFx/Nancy/pull/963)
- [Should be able to provide a root path provider instance]()
- [Changed scanning order](https://github.com/NancyFx/Nancy/pull/952)
- [User provided status code handlers aren't automatically discovered](https://github.com/NancyFx/Nancy/pull/952)
- [Caching Changes](https://github.com/NancyFx/Nancy/pull/941)
- [Removed overload of Dependencies that doesn't work](https://github.com/NancyFx/Nancy/pull/936)
- [Nancy module split](https://github.com/NancyFx/Nancy/pull/906)

## 0.15.3 ([View](https://github.com/NancyFx/Nancy/issues?milestone=21&state=closed))
- None

## 0.15.2 ([View](https://github.com/NancyFx/Nancy/issues?milestone=20&state=closed))
- [Url.Path needs to default to "/" if it's string.empty](https://github.com/NancyFx/Nancy/issues/911)

## 0.15.1 ([View](https://github.com/NancyFx/Nancy/issues?milestone=19&state=closed))
- [Open redirect in UserLoggedInRedirectResponse](https://github.com/NancyFx/Nancy/issues/893)

## 0.15 ([View](https://github.com/NancyFx/Nancy/issues?milestone=14&state=closed))
- [Changed discovery of ISerializer](https://github.com/NancyFx/Nancy/pull/869)
- [Nancy.Testing lovin'](https://github.com/NancyFx/Nancy/issues/633)
- [Limit the number of allowed multipart sections](https://github.com/NancyFx/Nancy/issues/518)

## 0.14.1 ([View](https://github.com/NancyFx/Nancy/issues?milestone=15&state=closed))
- None

## 0.14 ([View](https://github.com/NancyFx/Nancy/issues?milestone=12&state=closed))
- [Change IConvention detection](https://github.com/NancyFx/Nancy/pull/839)
- [Return 404 instread of diag help page in release](https://github.com/NancyFx/Nancy/pull/835)
- [Made the binder overwrite as default](https://github.com/NancyFx/Nancy/pull/834)
- [Renamed IErrorHandler to IStatusHandler](https://github.com/NancyFx/Nancy/pull/808) 

## 0.13 ([View](https://github.com/NancyFx/Nancy/issues?milestone=11&state=closed))
- [Made ConfigurableNancyModule parameters dynamic](https://github.com/NancyFx/Nancy/pull/781)
- [Re-ordered default view location and added new module path conventions](https://github.com/NancyFx/Nancy/pull/771)
- [ModulePath View Resolution not making sense](https://github.com/NancyFx/Nancy/issues/766)
- [View not found is broken in v0.12.1](https://github.com/NancyFx/Nancy/issues/759)
- [Moved TinyIoc namespace to Nancy.TinyIoc](https://github.com/NancyFx/Nancy/pull/724)

## 0.12.1 ([View](https://github.com/NancyFx/Nancy/issues?milestone=13&state=closed))
- [Negotiation returns xml instead of json](https://github.com/NancyFx/Nancy/issues/746)

## 0.12 ([View](https://github.com/NancyFx/Nancy/issues?milestone=10&state=closed))
- [Removed NDjango](https://github.com/NancyFx/Nancy/pull/729)
- [Diagnostics should be wired in after IApplicationStartup](https://github.com/NancyFx/Nancy/issues/692)
- [Exclude Nancy.Testing from auto-registration](https://github.com/NancyFx/Nancy/issues/647)
- [Update the Nancy.Testing.Fakes.FakeNancyModule](https://github.com/NancyFx/Nancy/issues/645)
- [Disable Auto Registration in ConfigurableBootstrapper](https://github.com/NancyFx/Nancy/issues/643)
- [Replace HtmlAgilityPack with CsQuery Improvement](https://github.com/NancyFx/Nancy/issues/640)
- [Split IStartup into IApplicationStartup and IApplicationRegistrations](https://github.com/NancyFx/Nancy/pull/635)
- [Reorder the view location conventions Improvement](https://github.com/NancyFx/Nancy/issues/632)
- [Split IStartup registration and initialization into two seperate concerns](https://github.com/NancyFx/Nancy/issues/569)

## 0.11 ([View](https://github.com/NancyFx/Nancy/issues?milestone=7&state=closed))
- [Added Nancy context to UserMapper](https://github.com/NancyFx/Nancy/pull/599)
- [Updated HtmlHelpers<T> and UrlHelpers<T>](https://github.com/NancyFx/Nancy/pull/592)
- [Remove version header from NancyEngine (Issue #489)](https://github.com/NancyFx/Nancy/pull/554)
- [Make the NancyContext available everywhere](https://github.com/NancyFx/Nancy/issues/550)
- [Add NancyContext as parameter in IErrorHandler.HandlesStatusCode](https://github.com/NancyFx/Nancy/issues/514)

## 0.10 ([View](https://github.com/NancyFx/Nancy/issues?milestone=6&state=closed))
- [Made helpers and url helper interfaces generic](https://github.com/NancyFx/Nancy/pull/516)
- [Improved the RequestHeaders class](https://github.com/NancyFx/Nancy/pull/476)
- [RequestHeaders.AcceptLanguage does not contain a weighed list of languages](https://github.com/NancyFx/Nancy/issues/474)
- [Configurable Bootstrapper Should Disable Error Handling](https://github.com/NancyFx/Nancy/issues/458)
- [View location conventions with module path doesn't work when containing leading /](https://github.com/NancyFx/Nancy/issues/453)
- [Should give better error messages when views not located](https://github.com/NancyFx/Nancy/issues/446)
- [Make date request headers nullable and better invalid date handling](https://github.com/NancyFx/Nancy/pull/403)
- [Tweaked request container logic. ](https://github.com/NancyFx/Nancy/pull/394)

## 0.9 ([View](https://github.com/NancyFx/Nancy/issues?milestone=9&state=closed))
- [Serializers should be "user specified then default"](https://github.com/NancyFx/Nancy/issues/365)
- [Change TypeConverter signature](https://github.com/NancyFx/Nancy/issues/364)
- [Make ErrorHandler a collection](https://github.com/NancyFx/Nancy/issues/361)
- [Removed url decoding in route matcher](https://github.com/NancyFx/Nancy/pull/359)
- [Basic auth for api](https://github.com/NancyFx/Nancy/pull/349)
- [Per-request application pipelines](https://github.com/NancyFx/Nancy/issues/345)

## 0.8.1 ([View](https://github.com/NancyFx/Nancy/issues?milestone=8&state=closed))
- [JavaScriptSerializer not working with TimeSpan](https://github.com/NancyFx/Nancy/issues/336)

## 0.8 ([View](https://github.com/NancyFx/Nancy/issues?milestone=4&state=closed))
- [Inject root path into GenericFileResponse](https://github.com/NancyFx/Nancy/issues/310)
- [Case sensitivity options Improvement](https://github.com/NancyFx/Nancy/pull/299)
- [Update HttpStatusCode enum to include additional codes](https://github.com/NancyFx/Nancy/issues/267)
- [Update the BrowserReponse class](https://github.com/NancyFx/Nancy/issues/243)
- [Update IStartup Improvement](https://github.com/NancyFx/Nancy/issues/237)
- [Give the FakeNancyBootstrapper some love](https://github.com/NancyFx/Nancy/issues/232)
- [Bind self host to multiple IP addresses](https://github.com/NancyFx/Nancy/issues/201)

## 0.7.1 ([View](https://github.com/NancyFx/Nancy/issues?milestone=5&state=closed))
- None

## 0.7 ([View](https://github.com/NancyFx/Nancy/issues?milestone=3&state=closed))
- [Route selection changes](https://github.com/NancyFx/Nancy/issues/194)
- [Cryptography Improvements](https://github.com/NancyFx/Nancy/issues/179)
- [Embedded views throws AmbiguousViewException](https://github.com/NancyFx/Nancy/issues/145)
- [Typed Properties for well-know Http Headers](https://github.com/NancyFx/Nancy/issues/127)

## 0.6 ([View](https://github.com/NancyFx/Nancy/issues?milestone=2&state=closed))
- None