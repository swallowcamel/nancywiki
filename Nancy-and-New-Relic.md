[New Relic](http://www.newrelic.com) is a nice tool used to monitor applications and collect performance metrics running on pretty much any platform, including .NET. Wiring up Nancy with [New Relic](http://www.newrelic.com) is a cinch! 

Out of the box, the NewRelic agent traces all transactions as `NancyHttpHandler` - not very useful! But with the [.NET Agent API](https://docs.newrelic.com/docs/agents/net-agent/features/net-agent-api), it's possible to set an explicit identifier for each transaction logged.

First, install the [corresponding NuGet package](http://www.nuget.org/packages/NewRelic.Agent.Api/). Then, add or edit a `Module` that will be the base for all your other modules:

```c#
using NewRelicAgent = NewRelic.Api.Agent.NewRelic; // protip: don't give class and namespace the same name. it's awkward.

public abstract class BaseModule : NancyModule
{
    public BaseModule()
    {
        Before += ctx =>
        {
            var routeDescription = ctx.ResolvedRoute.Description;
            NewRelicAgent.SetTransactionName("Custom", string.Format("{0} {1}", routeDescription.Method, routeDescription.Path));

            return null;
        };
    }
}
```
**Note:** I added the request method because it doesn't seem to be traced by NewRelic...

Now that transactions get more usable, it's also interesting to add Nancy's internals to NewRelic's transaction timings. To achieve that, create and deploy a [custom instrumentation file](https://docs.newrelic.com/docs/agents/net-agent/instrumentation/net-custom-instrumentation) with the following content:

```xml
<?xml version="1.0" encoding="utf-8"?>
<extension xmlns="urn:newrelic-extension">
    <instrumentation>
        <tracerFactory>
            <match assemblyName="Nancy" className="Nancy.NancyEngine">
                <exactMethodMatcher methodName="HandleRequest" />
            </match>
        </tracerFactory>
    </instrumentation>
</extension>
```
You will now see the time spent by Nancy (and your application code) to handle the request.