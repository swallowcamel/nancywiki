Nancy is a great choice for applications that want to run a lightweight HTTP server without the overhead of IIS. To do this, host it with WCF:

```c#
var host = new WebServiceHost(new NancyWcfGenericService(),
                              new Uri("http://localhost:1234/base/"));
host.AddServiceEndpoint(typeof (NancyWcfGenericService), new WebHttpBinding(), "");
host.Open();
            
// Nancy will now handle requests to http://localhost:1234/base/
```

Be sure that your project is using the full .NET 4.0 framework and that you have referenced both System.ServiceModel and System.ServiceModel.Web (for the WebServiceHost class).