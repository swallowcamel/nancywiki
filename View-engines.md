The concept of a view engine is quite simple - it takes a “template” and an optional “model” (the data) and outputs (usually) HTML to be rendered into the browser. By default, Nancy ships with a built in view engine known as the [`SuperSimpleViewEngine`](https://github.com/grumpydev/SuperSimpleViewEngine), which supports all the necessities such as layouts, partials, models, conditions and iterations. You can use this without any additional dependencies and it serves up `.html` and `.sshtml` files.

The following is an example of what a `.sshtml` file can look like

```html
    @Master['MasterPage']
    
    @Section['Content']
        <p>This content from the index page<p>
        <h3>Partials</h3>
        <p>Login box below rendered via a partial view with no model.</p>
        <div id="login">
            @Partial['login'];
        </div>
        <p>Box below is rendered via a partial with a sub-model passed in.</p>
        <p>The submodel is a list which the partial iterates over with Each</p>
        <div id="users">
            @Partial['user', Model.Users];
        </div>
        <h3>Encoding</h3>
        <p>Model output can also be encoded:</p>
        <p>@!Model.NaughtyStuff</p>
    @EndSection
```
In addition to the built in engine, Nancy also supports Razor, Spark, NDjango and the dotLiquid view engines as separate packages. By adding a reference to them from your project Nancy will automatically be able to render views using their supported file extensions.

## Rendering a view from a route action

To render a view you simply use the View indexer of the `NancyModule` with the appropriate parameters, which might sound complicated, but it really isn’t... honest! :-). There are three overloads which allow you to pass a view name, a model, or both and Nancy will take care of the rest for you, locating the view using a series of “conventions”, which we will discuss later:

    Get["/products"] = parameters => {
        return View["products.html", someModel];
    };

The view name offers you quite a bit of flexibility in terms of what information it can contain: 

1. Name of the view file with the file extension present (e.g. “products.html”)
2. Name of the view file without the file extension present (e.g. “products”). If there are two, or more, views with the same name, but with different extensions, you will receive an `AmbigiousViewsException` with details on which conflicting views were located.
3. A path, relative to the application root, where Nancy will look for the views (e.g. `products/products.html`). 

To learn more on where Nancy looks for views, and how to customize the logic, check out [[View location conventions]]

## Resolving the name of the view from the model

If you only pass in a model to the View indexer Nancy will attempt to infer the name and location of the view using another of the default conventions.

The convention strips “Model” off the end of the model type if it exists (e.g. ProductsModel becomes products), and then uses that name to attempt to locate the view. 

In the following route, Nancy will try to locate a view called Products with a file extension that matches any of the installed view engines:

    Get["/products"] = parameters => {
        return View[new ProductsModel()];
    };

[<< Part 7. Bootstrapper](Bootstrapper) - [Documentation overview](Documentation) - [Part 9. View location conventions >>](View location conventions)