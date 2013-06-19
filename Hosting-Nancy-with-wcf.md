Nancy is a great choice for applications that want to run a lightweight HTTP server without the overhead of IIS. To do this, take the following steps: 

* **Make sure you have the Nancy WCF hosting libary referenced - Install the package using Nuget**
```
PM> Install-Package Nancy.Hosting.Wcf
```

* **Then host it with WCF**

```c#
var host = new WebServiceHost(new NancyWcfGenericService(),
                              new Uri("http://localhost:1234/base/"));
host.AddServiceEndpoint(typeof (NancyWcfGenericService), new WebHttpBinding(), "");
host.Open();
            
// Nancy will now handle requests to http://localhost:1234/base/
```

Be sure that your project is using the full .NET 4.0 framework and that you have referenced both System.ServiceModel and System.ServiceModel.Web (for the WebServiceHost class).