Nancy is a great choice for applications that want to run a lightweight HTTP server without the overhead of IIS. To do this, take the following steps.

Create your **paket.references** file for a new F# Console App:

```
FSharp.Core
Nancy.Owin
```

Run `paket install` and then; add an implementation to your console app:

``` fsharp
namespace Suave.NancyFx

open Suave.Types
open Suave.Web
open Suave.Owin
open Nancy
open Nancy.Owin

type App() as x =
  inherit NancyModule()
  do
    x.Get.["/"] <- fun _ -> "Hello World, from NancyFx on Suave!" :> obj
    x.Get.["/fsharp"] <- fun _ -> "I can into F#" :> obj
    x.Get.["/json"] <- fun _ -> x.Response.AsJson([ "Test" ]) :> obj
    x.Get.["/complex"] <- fun _ -> 
      let response = x.Response.AsJson(["This is my Response"])
      response.ContentType <- "application/json"
      response.Headers.Add("Funky-Header", "Funky-Header-Value")
      response :> obj

module Program =

  [<EntryPoint>]
  let main argv =
    let opts = new NancyOptions()
    let app = OwinApp.ofMidFunc (NancyMiddleware.UseNancy(opts))
    startWebServer defaultConfig app
    0
```

Run your app!

``` bash
$ curl -i http://localhost:8083
HTTP/1.1 200 OK
Server: Suave/0.30.0.0 (http://suave.io)
Date: Mon, 31 Aug 2015 08:23:25 GMT
Content-Type: text/html
Content-Length: 35

Hello World, from NancyFx on Suave!%
```