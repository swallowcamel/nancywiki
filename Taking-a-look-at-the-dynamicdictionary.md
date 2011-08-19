The `DynamicDictionary` is a dynamic type with dictionary-like behaviors, but there are a lot more to it under the covers and its a big part of what helps Nancy provide a terse syntax in many places. The most obvious place, is in the lambda expression that forms the route action. Any captured values from the request will be put into a DynamicDictionary and passed into the action for the route.

With the DynamicDictionary you are able to access the values using a standard indexer approach or by using the name, of the captured, value just as a normal property. Both produce the same result, it all comes down to personal preference.

    Get[“/hello/{name}”] = parameters => {
        return “Hello “ + parameters.name;
    };

    Get[“/goodbye/{name}”] = parameters => {
        return “Goodbye “ + parameters[“name”];
    };
	
The stored values can explicitly or implicitly be cast to most of the primitive types and a special property, called `HasValue` can be checked to determine if a value has been assigned or not. The values also implement the `IEquatable<>` and `IConvertible` interfaces.