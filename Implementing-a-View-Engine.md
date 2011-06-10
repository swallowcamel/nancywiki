_This page will contain implementation advice for view engines_

* View engines are singletons, do not store state in them
* If you add dependencies to your engine, add a default ctor with default configuration
* Make use of the view cache on the render context that is passed into the IViewEngine.RenderView method