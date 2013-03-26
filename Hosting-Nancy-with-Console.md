You can use a console app to host Nancy.  Simply use the code below to host Nancy.

    var nancyHost = new Nancy.Hosting.Self.NancyHost(new Uri("http://localhost:1234"));
    nancyHost.Start();

    Console.ReadLine();
    nancyHost.Stop();

You can then define your Modules etc and they will get hit as expected