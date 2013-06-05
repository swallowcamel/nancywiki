Content negotiation is a way of determining what content type to return to the client based on what the client can handle, and what the server can provide. Whenever you return something from a route, that is not a `Response` object (or anything that can implicitly be cast to a `Response` object) then that will be passed through the content negotiation pipeline before being sent back in the response.

```c#
Get["/"] = parameters => {
  return new MyModel();
};
```

The content negotiation pipeline will inspect the incoming `Accept` headers and determine which of the requested media types is the most suitable and format the response accordingly.

## The response processors

Processors are used to determine the most suitable content type to return to the client, and are used to actually convert the model to the corresponding type.

To create a response processor you simply implement the `IResponseProcessor` interface

```c#
public interface IResponseProcessor
{
    /// <summary>
    /// Gets a set of mappings that map a given extension (such as .json)
    /// to a media range that can be sent to the client in a vary header.
    /// </summary>
    IEnumerable<Tuple<string, MediaRange>> ExtensionMappings { get; }

    /// <summary>
    /// Determines whether the the processor can handle a given content type and model.
    /// </summary>
    ProcessorMatch CanProcess(MediaRange requestedMediaRange, dynamic model, NancyContext context);

    /// <summary>
    /// Process the response.
    /// </summary>
    Response Process(MediaRange requestedMediaRange, dynamic model, NancyContext context);
}
```

The response processors are automatically discovered and wired up at runtime and will be invoked in the order of which they were discovered. If you need more explicit control over the order, in which they are invoked, then you should explicitly set the `ResponseProcessors` member of the `NancyInternalConfiguration` in your bootstrapper.

```c#
public class Bootstrapper : DefaultNancyBootstrapper
{
    protected override NancyInternalConfiguration InternalConfiguration
    {
        get
        {
            var processors = new[]
            {
                typeof(SomeProcessor),
                typeof(AnotherProcessor)
            };

            return NancyInternalConfiguration.WithOverrides(x => x.ResponseProcessors = processors);
        }
    }
}
```

It should be noted that because the response processors are registered in the bootstrapper, they can take advantage of the fact and take whatever constructor dependencies it desires. 

### Match prioritization 

When a response is about to be converted into the format of the requested media type, then Nancy will query the `CanProcess` method of all available processors and aggregate the `ProcessorMatch` return values.

The `ProcessorMatch` type enables each processor to let Nancy know how well of a fit it is to process the requested media type and the type of the model that is being processed.

```c#
public class ProcessorMatch
{
    /// <summary>
    /// Gets or sets the match result based on the content type
    /// </summary>
    public MatchResult RequestedContentTypeResult { get; set; }

    /// <summary>
    /// Gets or sets the match result based on the model
    /// </summary>
    public MatchResult ModelResult { get; set; }
}
```

The `MatchResult` enumeration specified how well of a match the media type and model are. 

```c#
public enum MatchResult
{
    /// <summary>
    /// No match, nothing to see here, move along
    /// </summary>
    NoMatch,

    /// <summary>
    /// Will accept anything
    /// </summary>
    DontCare,

    /// <summary>
    /// Matched, but in a non-specific way such as a wildcard match or fallback
    /// </summary>
    NonExactMatch,

    /// <summary>
    /// Exact specific match
    /// </summary>
    ExactMatch
}
```

Once it has all the `ProcessorMatch` return values it orders them, in decending order, by the `ModelResult` followed by the `RequestedContentTypeResult`. This means that the processor with the best match for both is the one that will be used to format the model. If there happens to be two or more processors with the same match then Nancy will select one of them, but which one is not guarateed.

### Default response processors

Nancy is shipped with a couple of default response processors that are automatically available to you in your applications

- `JsonProcessor` - Converts the return value into json, when the requested media type is `application/json` or any vendor specific json media type `application/vnd.foobar+json`
- `ViewProcessor` - Renders a view, using the return value as the model, when the requested media type is `text/html`. It uses the normal [[View location conventions]], when choosing which view to render.
- `XmlProcessor` - Converts the return value into xml, when the requested media type is `application/xml` or any vendor specific json media type `application/vnd.foobar+xml`

## Controlling the negotiation

By returning a model from your route, you are giving Nancy free reign to negotiate the format of the response using all the available resources. But what if you want a bit more control, on a per-route basis, on how the negotiation process took place?

That is where the `Negotiator` comes into place. The `Negotiator` (you find it in `Nancy.Responses.Negotiation`) is type that you can return, from your route, and which contains instructions for the content negotiation pipeline. 

The `Negotiatior` has a single property on it, `NegotiationContext`, which if of a type with the same name. Using the `NegotiationContext` you can provide the instructions that control the negotiation of the response. 

However, directly working with the `Negotiator` and `NegotiationContext` can be a bit cumbersome and for that reason the `NancyModule` contains a helper called `Negotiate`, which exposes a bit more user-friendly API for constructing `Negotiator` instance.

This is an example of what it can look like when using the `Negotiator` from a route

```c#
Get["/"] = parameters => {
    return Negotiate
        .WithModel(new RatPack {FirstName = "Nancy "})
        .WithMediaRangeModel("text/html", new RatPack {FirstName = "Nancy fancy pants"})
        .WithView("negotiatedview")
        .WithHeader("X-Custom", "SomeValue");
};
```

The `Negotiator` contains several methods that you use to configure the returned `Negotiator` instance.
- `WithHeader` - Adds a header and value which should be attached to the response before being sent back to the client
- `WithHeaders` - Adds a collection of headers and values which should be attached to the response before being sent back to the client
- `WithView` - Enables you to be explicit with the name of the view which should be used when the client requests the `text/html` media type
- `WithModel` - Defines the model that should be used during the negotiation.
- `WithMediaRangeModel` - Enables you to define media type specific models that should be used during negotiation. The negotiation pipeline will first attempt to find a media type specific model to use and will fallback to the default model, which is specified using `WithModel`, if it fails.
- `WithFullNegotiation` - A helper to set the permissible media range to `*/*`
- `WithAllowedMediaRange` - Specifies a media range that should be permitted to be negotiated. This defaults to “*/*”, but the wildcard is removed as soon a specific content type (or types) is given.
- `WithStatusCode` - The status code that should be added to the response once it has been negotiated

## File extension support

Nancy supports extension-based access to “hotwire” the negotiation process, bypassing the normal accept headers. 

For example

```c#
Get["/ratpack"] = parameters => {
    return new RatPack {FirstName = "Nancy "});
};
```

Could be invoked by requesting `/ratpack` and an accept header of `application/json` and Nancy would attempt to convert the `RatPack` object into json before returning to the client. However, you could also request `/ratpack.json`, and not send in the accept header `application/json` and you would end up with the same result.

Internally, Nancy will detect the extension and query the available response processor’s `ExtensionMappings` property and see if any of them supports the extension. If there is a match then it will take the mapped media type and add it to the list of accept headers, but with a quality (1.1) which is higher than the other available accept headers.
It will then strip the extension from the request and attempt to find a matching route. If it fails to find a matching route it will append the file extension again, remove the added accept header, and attempt to process it as normal.

## Accept header coercion

Unfortunately certain browsers and javascript frameworks do not send valid accept headers, which could cause content negotiation to return a non-deal content type. To work around this the negotiation pipeline has a concept of “accept header coercion” that looks at the request and tweaks the accept headers to try and smooth things out.

The format of the convention is:

```c#
Func<
   IEnumerable<Tuple<string, decimal>>,
   NancyContext,
   IEnumerable<Tuple<string, decimal>>>
```

Essentially this is a function that takes the `NancyContext` and the current accept headers (name and quality pairs) and expects you to return the modified list of accept headers.

By default Nancy ships with the following conventions, in the  `Nancy.Conventions.BuiltInAcceptHeaderCoercions class`,  with those marked with (*) switched on by default:

- `BoostHtml` (*) - boosts the priority of text/html if it’s specified by has a lower priority than other content types.
- `CoerceBlankAcceptHeader` (*) - proves a default accept header if a blank one is specified.
- `CoerceStupidBrowsers` - replaces the accept header for older browsers that say they want xml rather than html.

Altering which (if any) coercions are enabled is done via the `ConfigureConventions` method in the bootstrapper:

```c#
public class Bootstrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureConventions(NancyConventions nancyConventions)
    {
        base.ConfigureConventions(nancyConventions);

        this.Conventions.AcceptHeaderCoercionConventions.Add((acceptHeaders, ctx) => {
                
            // Modify the acceptHeaders by adding, removing or updating the current
            // values.

            return acceptHeaders;
        });
    }
}
```

You could of course inherit the bootstrapper type that you are using in your project

## Defining your own conventions using IConventions

You can also create a class that implements the `IConventions` interface and in the `Initialise` method you add you conventions to the `AcceptHeaderCoercionConventions` property of the conventions that are passed in.

Nancy will locate all implementation of the interface and wire up the conventions, before they are passed onto the `ConfigureConventions` method of the bootstrapper.

## Automatic negotiation headers

Nancy will automatically add link and vary headers to negotiated responses. The link headers link to other representations based on file extension (e.g. .json.xml etc), and the vary header is to let caches know that the contents of the response will vary depending on the accept header the client provides.

## More Info
* [[Nancy and Content Negotiation|http://www.philliphaydon.com/2012/11/nancy-and-content-negotiation/]]
* [[Revisting Content Negotiation and APIs part 1|http://www.philliphaydon.com/2013/04/nancyfx-revisiting-content-negotiation-and-apis-part-1/]]
* [[Revisting Content Negotiation and APIs part 2|http://www.philliphaydon.com/2013/05/nancyfx-revisiting-content-negotiation-and-apis-part-2/]]
* [[Revisting Content Negotiation and APIs part 3|http://www.philliphaydon.com/2013/05/nancyfx-revisiting-content-negotiation-and-apis-part-3/]]

[<< Part 18. The cryptography helpers](The cryptography helpers) - [Documentation overview](Documentation) - [Part 20. The Super Simple View Engine >>](The Super Simple View Engine)