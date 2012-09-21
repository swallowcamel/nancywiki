[New Relic](http://www.newrelic.com) is a nice tool used to monitor applications and collect performance metrics running on pretty much any platform, including .NET. Wiring up Nancy with [New Relic](http://www.newrelic.com) is a cinch! Be sure to [install the appropriate packages](http://nuget.org/packages?q=NewRelic) first. In the following example, we identify each new Relic transaction as the HTTP Method and the Route. Otherwise all transactions show up as NancyHttpHandler - not very useful! We also log all exceptions via the error pipeline.

```c#
    using NewRelicAgent = NewRelic.Api.Agent.NewRelic; // protip: don't give class and namespace the same name. it's awkward.
    public class NewRelicStartup : IApplicationStartup
    {
        private readonly IRouteResolver routeResolver;

        public NewRelicStartup(IRouteResolver routeResolver)
        {
            this.routeResolver = routeResolver;
        }

        public void Initialize(IPipelines pipelines)
        {
            pipelines.BeforeRequest.AddItemToStartOfPipeline(
                context =>
                    {
                        var route = routeResolver.Resolve(context);
                        if (route == null || route.Item1 == null || route.Item1.Description == null) // probably not necessary but don't want the chance of losing visibility on anything
                        {
                            NewRelicAgent.SetTransactionName(
                                context.Request.Method, 
                                context.Request.Url.ToString());
                        }
                        else
                        {
                            NewRelicAgent.SetTransactionName(
                                route.Item1.Description.Method,
                                route.Item1.Description.Path);
                        }
                        return null;
                    });

            pipelines.OnError.AddItemToEndOfPipeline(
                (context, ex) =>
                    {
                        NewRelicAgent.NoticeError(
                            ex);
                        return null;
                    });
        }
    }
```