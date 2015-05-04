The `DynamicDictionary` is a dynamic type with dictionary-like behaviors, but there is a lot more to it under the covers and it's a big part of what helps Nancy provide a terse syntax in many places. The most obvious place, is in the lambda expression that forms the route action. Any captured values from the request will be put into a DynamicDictionary and passed into the action for the route.

With the DynamicDictionary you are able to access the values using a standard indexer approach or by using the name, of the captured, value just as a normal property. Both produce the same result, it all comes down to personal preference.

```c#
Get["/hello/{name}"] = parameters => {
    return "Hello " + parameters.name;
};

Get["/goodbye/{name}"] = parameters => {
    return "Goodbye " + parameters["name"];
};
```
	
It is worth noting that if you prefer accessing captured properties via a normal property you must assign the capture group a valid C# identifier (i.e.: one that starts with an alphabetic character). There's no restriction on the name of the DynamicDictionary instance specified in your handler signature, but Nancy borrows a convention from functional programming whereby the underscore (_) indicates the value is not used:

```c#
// Underscore indicates that our method doesn't use the parameters object. 
Get["/hello"] = _ => {
    return "Hello, World!";
};

// whereas in this method we're using the DynamicDictionary so we give it a proper name.
Get["/hello/{name}"] = parameters => {
    return "Hello" + parameters.name;
};
```

The stored values can explicitly or implicitly be cast to most of the primitive types and a special property, called `HasValue` can be checked to determine if a value has been assigned or not. The values also implement the `IEquatable<>` and `IConvertible` interfaces.

[<< Part 4. Async](Async) - [Documentation overview](Documentation) - [Part 6. The before and after module hooks >>](The before and after module hooks)