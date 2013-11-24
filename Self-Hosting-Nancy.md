With the [`Nancy.Hosting.Self`](https://nuget.org/packages/Nancy.Hosting.Self) NuGet package you can self-host Nancy in, for example, console application or embedded in your desktop or windows service application.

After you have installed the package, create an instance of the `NancyHost` class and call the `Start` method.

    using (var host = new NancyHost(new Uri("http://localhost:1234"))
    {
       host .Start();
       Console.ReadLine();
    }

`NancyHost` implements the `IDisposable` interface and will automatically call the `Stop` method on the host when the application is closing. If you need more control over when the host is started and stopped, simply create an instance, without the using-statement, and explicitly call `Start()` and `Stop()` as needed.

## Host Configuration

When creating an instance of `NancyHost` you have the ability to pass in a configuration object to customize the behavior of your host. 

Below is a list of available configuration options, provided by the `HostConfiguration` type.

Property|Description|Default
--------|-----------|-------
AllowChunkedEncoding|Determines if `Transfer-Encoding: Chunked` is allowed for the response instead of `Content-Length`|true
EnableClientCertificates|Determines whether client certificates are enabled or not. When set to true the host will request a client certificate if the request is running over SSL|false
RewriteLocalhost|Determines if localhost uris are  rewritten to `http://+:port/` style uris to allow for listening on all ports, but requiring either a namespace reservation, or admin access|true
UnhandledExceptionCallback|Provides a callback to be called if there's an unhandled exception in the host. __Note:__ this will __not__ be called for normal Nancy exceptions that are handled by the Nancy handlers. The callback is of the type `Action<Exception>`|Write to debug window
UrlReservations|Configuration around automatically creating namespace reservations|See below

The `UrlReservations` property provides several options that can be used to determine the behavior

Property|Description|Default
--------|-----------|-------
CreateAutomatically|Value indicating whether url reservations are automatically created when necessary|false
User|Value for the user to use to create the url reservations for|The `Everyone` group

## Namespace Reservations

Windows requires applications to register the part of the HTTP URL namespace, that they will be, listening on, or for the application to run with administrator privileges. This is show the MSDN documentation describes the requirement

> Namespace reservation assigns the rights for a portion of the HTTP URL namespace to a particular group of users. A reservation gives those users the right to create services that listen on that portion of the namespace. Reservations are URL prefixes, meaning that the reservation covers all sub-paths of the reservation path.

Failing to do so is going to result in an access denied, from the operating system, when the application tries to open and listen on a certain url and port. In the Nancy self-host this will result in an `AutomaticUrlReservationCreationFailureException` being thrown with a detailed error message, that contains information on how to resolve it.

Below is the message of the exception that was thrown for an application that tried to run Nancy on the following URLs

- http://localhost:8799/nancy/
- http://localhost:9987/nancy/

and with `RewriteLocalhost` setting set to `true` (see __Host Configuration__ for more information about the 
`RewriteLocalhost` setting)

> The Nancy self host was unable to start, as no namespace reservation existed for the provided url(s).
>
> Please either enable UrlReservations.CreateAutomatically on the HostConfiguration provided to 
> the NancyHost, or create the reservations manually with the (elevated) command(s):
>
> netsh http add urlacl url=http://+:8799/nancy/ user=Everyone
>
> netsh http add urlacl url=http://+:9987/nancy/ user=Everyone

Replace `DOMAIN\username` with your domain and username or your computer name and username if you are not joined to a domain. See <http://msdn.microsoft.com/en-us/library/ms733768.aspx> for more information.