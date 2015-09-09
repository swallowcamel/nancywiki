A blog post explaining this in C# is available [here](http://mike-ward.net/2013/04/11/custom-error-pages-in-nancyfx/)

Additional (more recent) C# post is available [here](https://blog.tommyparnell.com/custom-error-pages-in-nancy/).

A blog post in VB.Net is available [here](http://blogs.lessthandot.com/index.php/WebDev/ServerProgramming/nancy-and-custom-error-pages)

**A note on the IStatusCodeHandler in release mode**

If your app is setup with `IStatusCodeHandler`s, and you want to continue to provide custom error pages for status codes of 50x (internal server error) for view rendering exceptions when compiling for release, you will need to manually set StaticConfiguration.DisableErrorTraces = false in your Bootstrapper.ApplicationStartup override. For more info on this, see issue #2052.

[<< Part 17. Adding a custom FavIcon](Adding a custom FavIcon) - [Documentation overview](Documentation) - [Part 19. The cryptography helpers >>](The cryptography helpers)