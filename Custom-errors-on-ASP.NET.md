I wanted to be able to return custom errors from a Nancy (0.9) app hosted in ASP.NET. Turns out that's easier said than done.

## Get ASP and IIS out of your way

Add to or change your `Web.config`:

```xml
<configuration>
  <system.webServer>
    <httpErrors errorMode="Custom" existingResponse="PassThrough" />
  </system.webServer>
</configuration>
```

With this change, you should now see the default Nancy error pages. If that's all you want, excellent, but I wanted to return error messages straight from a route handler, so on we go!

## Handy response extension

Pop this extension method somewhere:

```c#
public static Response AsError(this IResponseFormatter formatter, HttpStatusCode statusCode, string message)
{
    return new Response
        {
            StatusCode = statusCode,
            ContentType = "text/plain",
            Contents = stream => (new StreamWriter(stream) { AutoFlush = true }).Write(message)
        };
}
```

Now in your route handlers you can simply

```c#
Get["/"] = parameters => Response.AsError(HttpStatusCode.PaymentRequired, "pony up the dough");
```

But you'll still see the default Nancy error page! One more thing...

## Have Nancy step aside

In your bootstrapper, you'll need to override the default error handler with your own. Here's what I did:

```c#
protected override NancyInternalConfiguration InternalConfiguration
{
    get
    {
        return NancyInternalConfiguration.WithOverrides(
            builder => builder.ErrorHandlers = new List<Type>() {
                typeof(NullErrorHandler)
            });
    }
}
```

Then implement a `NullErrorHandler` class that inherits from IErrorHandler and handles all status codes by doing exactly nothing.

Now you'll see

```bash
% curl http://localhost/
pony up the dough
```