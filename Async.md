## .NET 4.5 Targeting

If you're getting the error:

`Operator '+=' cannot be applied to operands of type 'Nancy.BeforePipeline' and 'lambda expression'
Cannot convert lambda expression to type 'Nancy.BeforePipeline' because it is not a delegate type`

Confirm that your project is targeting .NET 4.5 before starting to use the async/await keywords. 

## Syntax

Both the Before/After pipelines, and the main route delegates, can be specified as async. The syntax is *almost* identical to the sync code, but with the following changes:

* The before and after hooks take two parameters, context and a cancellation token, rather than just the context.
* The route definition has an additional boolean parameter, and the delegate takes two parameters, the captured params, and the cancellation token.

## Syntax Example

```c#
public MainModule()
{
    Before += async (ctx, ct) =>
        {
            this.AddToLog("Before Hook Delay\n");
            await Task.Delay(5000);

            return null;
        };

    After += async (ctx, ct) =>
        {
            this.AddToLog("After Hook Delay\n");
            await Task.Delay(5000);
            this.AddToLog("After Hook Complete\n");

            ctx.Response = this.GetLog();
        };

    Get["/", true] = async (x, ct) =>
        {
            this.AddToLog("Delay 1\n");
            await Task.Delay(1000);

            this.AddToLog("Delay 2\n");
            await Task.Delay(1000);

            this.AddToLog("Executing async http client\n");
            var client = new HttpClient();
            var res = await client.GetAsync("http://nancyfx.org");
            var content = await res.Content.ReadAsStringAsync();

            this.AddToLog("Response: " + content.Split('\n')[0] + "\n");

            return (Response)this.GetLog();
        };
}
```

## Hot Tasks / 4.0 Support

While there is nothing in the async support that *requires* 4.5 async/await (it's not used internally and still targets .net 4), we *do* expect that Tasks returned by routes to be "hot" (already started) If you choose to use this using only the TPL or using RX, please ensure every task that's returned from the delegates is hot, or the request will likely never return.

[<< Part 3. Legacy Routing](Legacy Routing) - [Documentation overview](Documentation) - [Part 5. Taking a look at the DynamicDictionary >>](Taking a look at the DynamicDictionary)