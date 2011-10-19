_This page will contain implementation advice for view engines_

* Name the engine xxxViewEngine, where xxx is the name of the parser, such as Razor, Spark and so on
* The only public methods on a view engine should be those of the IViewEngine interface
* View engines are singletons, do not store state in them
* If you add dependencies to your engine, add a default ctor with default configuration
* Make use of the view cache on the render context that is passed into the IViewEngine.RenderView method - **but** make sure that the item you cache does not contain state that could end up being reused between multiple requests
* The initialize method is called when the view engine is created and all views that the engine can render will be passed in, along with the view cache
* If possible, integrate support for csrf (cross-site request forgery). Nancy comes with built in support but the engine needs to make use of it