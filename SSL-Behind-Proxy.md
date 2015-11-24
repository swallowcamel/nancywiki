#SSL Behind Proxy

Sometimes you may be in a position where your Nancy app is behind a proxy eg/nginx which uses SSL.  When the proxy routes the request to Nancy the request scheme will be http.  To make it match the proxy scheme you can use `SSLProxy.RewriteSchemeUsingForwardedHeaders`.

In your Bootstrapper simply call `SSLProxy.RewriteSchemeUsingForwardedHeaders(pipelines);` in `ApplicationStartup/RequestStartup` and you're good to go.

**NOTE** Your proxy must pass in either the `X-Forwarded-Proto` header or the newer `Forwarded` header with the value `proto=https` inside it

[<< Part 22. Token](Token Authentication) - [Documentation overview](Documentation)