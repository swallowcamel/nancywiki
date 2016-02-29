Nancy has localization built into it.  

There are a set of [conventions][1] on how to determine the current culture and also a set of view [conventions][2] based on culture. 

So if culture is `de-DE` it will look for a view called `Home-de-DE`, which you can then add your localized content.  

It will also use resx files eg. `Text.resx`, `Text.de-DE.resx` (but can be [overridden][3] to be a database or whatever) to get translations out so you can in your view call `@Text.Greeting` and it will find the relevant item.

[Here][4] is a Razor demo showing Localization

[1]: https://github.com/NancyFx/Nancy/blob/master/src/Nancy/Conventions/DefaultCultureConventions.cs
[2]: https://github.com/NancyFx/Nancy/blob/master/src/Nancy/Conventions/DefaultViewLocationConventions.cs
[3]: https://github.com/NancyFx/Nancy/blob/master/src/Nancy/Localization/ResourceBasedTextResource.cs
[4]: https://github.com/NancyFx/Nancy/tree/master/samples/Nancy.Demo.Razor.Localization

[<< Part 11. View location conventions](View location conventions) - [Documentation overview](Documentation) - [Part 13. Testing your application >>](Testing your application)