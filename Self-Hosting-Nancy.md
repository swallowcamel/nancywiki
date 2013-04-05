# Self Hosting Nancy

You can use a console app to host Nancy.  Simply install the [`Nancy.Hosting.Self`](https://nuget.org/packages/Nancy.Hosting.Self) NuGet package and use the code below to host Nancy.

    var nancyHost = new Nancy.Hosting.Self.NancyHost(new Uri("http://localhost:1234"));
    nancyHost.Start();

    Console.ReadLine();
    nancyHost.Stop();

You can then define your Modules etc and they will get hit as expected.

## HttpListenerException
Note that on Windows hosts a `HttpListenerException` may be thrown with an `Access Denied` message. To resolve this the URL has to be added to the ACL. Execute the following in PowerShell or CMD running as administrator:

    netsh http add urlacl url=http://+:1234/ user=DOMAIN\username

Replace `DOMAIN\username` with your domain and username or your computer name and username if you are not joined to a domain. See <http://msdn.microsoft.com/en-us/library/ms733768.aspx> for more information.

Also this should be obvious but the port may need to be opened on the machine or corporate firewall to allow access to the service.