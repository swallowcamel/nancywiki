The default serialization provided by Nancy for JSON data should in most instances automatically do the right thing. Reflection is used to obtain a list of data members in the object, and then each data member's value is converted to the correct type and assigned. If the data member's type is itself a class type, then the process is used, recursively, for that object as well.

Sometimes, though, there might be circumstances when you need more than this simple algorithm to translate data.

## Custom representation of class types

Consider, for instance, if your external interface had a requirement that a date needed to be specified as its individual components, year, month and day:

```json
{"orderDate":{"year":2014,"month":7,"day":12}}
```

You *could* just make a data type that stored ```year```, ```month``` and ```day``` fields, but then every time you needed to treat the order date as a ```DateTime```, you'd have to convert the values manually. The serialization system in Nancy allows you to do better than this: Using a specialized ```JavaScriptConverter```, you can intercept the serialization and deserialization of ```DateTime``` values and emit the data in any format you want. The way a ```JavaScriptConverter``` works saves you from having to directly manipulate JSON data; rather than JSON text, the serialized form is an associative array, an ```IDictionary<string, object>```. When deserializing, the JSON text is converted to this form before any other conversion is done. When serializing, this dictionary is then translated by Nancy into the text representation of a JSON object, and whatever objects you place into the array go back into the start of the serializer and can themselves be handled by converters.

To create a ```JavaScriptConverter```, you need to do two things:

1. You need to indicate to the serialization system what data types your converter will be handling.
2. You need to provide the code that translates between your data type and ```IDictionary<string, object>```.

These steps are done by implementing the abstract members of the ```JavaScriptConverter``` base class.

To indicate which data types your converter supports, you need to supply an implementation of the ```SupportedTypes``` property. This property returns an ```IEnumerable<Type>```, which means a simple implementation can look like this:

```c#
  public override IEnumerable<Type> SupportedTypes
  {
    get { yield return typeof(DateTime); }
  }
```

A more complicated converter supporting multiple types could store the list in a static array and return that array, since arrays implement ```IEnumerable`1``` as well.

To perform the actual conversion, supply implementations of ```Serialize``` and ```Deserialize```:

```c#
  public override IDictionary<string, object> Serialize(object obj, JavaScriptSerializer serializer)
  {
    if (obj is DateTime)
    {
      DateTime date = (DateTime)obj;

      var json = new Dictionary<string, object>();

      json["year"] = date.Year;
      json["month"] = date.Month;
      json["day"] = date.Day;

      return json;
    }

    return null;
  }

  public override object Deserialize(IDictionary<string, object> json, Type type, JavaScriptSerializer serializer)
  {
    if (type == typeof(DateTime))
    {
      object year, month, day;

      json.TryGetValue("year", out year);
      json.TryGetValue("month", out month);
      json.TryGetvalue("day", out day);

      if ((year is int)
       && (month is int)
       && (day is int))
        return new DateTime((int)year, (int)month, (int)day);
    }

    return null;
  }
```

You can choose to do as much or as little conversion as you want. This sample, for instance, is pretty strict about the fields it expects to see; they must be ```int``` values. It is not strict about whether there are other fields present, though. A stricter implementation might refuse to translate a JSON object with extra properties, while a less strict implementation might accept and attempt to convert non-integer values. ```JavaScriptConverter``` allows you to tailor the serialization and deserialization processes to your exact needs.

[[View Complete Sample Class|Sample JavaScriptConverter]]

## Custom representation of individual values

There are some instances where you might want to control the conversion of an object directly to/from a primitive JSON value, rather than a JSON object. For instance, if your models include byte arrays, you might want to serialize these as Base 64 strings. The default serializer will read and write JSON text like ```[72,101,108,108,111,44,32,119,111,114,108,100]```. The corresponding Base 64 representation of this is ```SGVsbG8sIHdvcmxk```, which is significantly shorter and also faster to parse. You could simply declare the data field as a ```string``` in your model, but it is possible to make this conversion take place seamlessly during the serialization and deserialization processes using a ```JavaScriptPrimitiveConverter```.

A ```JavaScriptPrimitiveConverter``` is quite similar to a ```JavaScriptConverter```, except that the JSON representation is simply an ```object```, rather than a full ```IDictionary<string, object>```. Whatever value you return in that ```object``` will be written directly to JSON output during serialization. When deserializing, if the data type for the corresponding field in your model type is matched by a ```JavaScriptPrimitiveConverter```, then the primitive value is handed over to the converter, which may return any C# object it desires.

The steps for implementing a ```JavaScriptPrimitiveConverter``` are the same as those for creating a ```JavaScriptConverter```:

1. You need to indicate to the serialization system what data types your converter will be handling.
2. You need to provide the code that translates between your data type and ```object```.

To indicate which data types your converter supports, supply an implementation of the ```SupportedTypes``` property. See the description above of the ```JavaScriptConverter``` property by the same name for more information.

To perform the actual conversion, supply implementations of ```Serialize``` and ```Deserialize```:

```c#
  public override IEnumerable<Type> SupportedTypes
  {
    get { yield return typeof(byte[]); }
  }

  public override object Serialize(object obj, JavaScriptSerializer serializer)
  {
    var byteArray = obj as byte[];

    if (byteArray != null)
    {
      try
      {
        return Convert.ToBase64String(byteArray);
      }
      catch { }
    }

    return null;
  }

  public override object Deserialize(object jsonPrimitive, Type type, JavaScriptSerializer serializer)
  {
    if ((type == typeof(byte[])) && (jsonPrimitive is string))
    {
      try
      {
        return Convert.FromBase64String(jsonPrimitive as string);
      }
      catch { }
    }

    return null;
  }
```

Another possible use of a ```JavaScriptPrimitiveConverter``` might be to support ```XmlElement``` fields embedded in model types. By default, ```XmlElement``` instances won't produce any output with the JSON serializer, because the data type does not have any data members that can be both read and written to. However, a ```JavaScriptPrimitiveConverter``` could take over handling of ```XmlElement``` values and convert the subtrees they represent into the corresponding XML text, to be output as a JSON string for transport.

[[View Complete Sample Class|Sample JavaScriptPrimitiveConverter]]

## How do I actually use them?

```JavaScriptConverter```s and ```JavaScriptPrimitiveConverter```s must be enabled in each ```JavaScriptSerializer``` object in order to be active. There are two ways to accomplish this:

1. If you are creating a ```JavaScriptSerializer``` for a specific task within your code, you can explicitly register converters using the ```RegisterConverters``` method. This method allows a series of ```JavaScriptConverter``` and/or ```JavaScriptPrimitiveConverter``` objects to be supplied.

2. If you need to use the converters in a situation where the actual ```JavaScriptSerializer``` object is being implicitly created, or if you need the effects of a converter to be available globally, you can add converters to the static collections ```JsonSettings.Converters``` and ```JsonSettings.PrimitiveConverters```. The ```JavaScriptSerializer``` objects used within model binding and default model serialization by Nancy automatically register any converters found in these collections at the time they are constructed.

## What about XML?

Nancy uses the .NET Framework's own built-in ```XmlSerializer``` infrastructure to handle clients sending and receiving data using XML as the transport format. The design of ```XmlSerializer``` is quite extensible, and the way in which it is extensible is different from the ```JavaScriptConverter``` and ```JavaScriptPrimitiveConverter``` types that JSON serialization employs. ```XmlSerializer``` is unaware of JSON converters, and ```JavaScriptSerializer``` ignores XML-specific attributes. Thus, extensions to XML serialization and to JSON serialization can coexist in the same project without interfering with one another.

It is beyond the scope of this documentation to fully explain how to take control of ```XmlSerializer```'s serialization & deserialization processes, but the effects in the sample converters above can be handled with XML in two ways:

1. The ```[XmlIgnore]``` attribute can be supplied to ensure that a field whose data type cannot be handled by the default serialization is skipped. The data within that field can then be formatted in any way you want by the use of a second property. Use a ```[ScriptIgnore]``` attribute to prevent this second property from appearing in JSON output. This property can also be made invisible within the IDE using the ```[EditorBrowsable]``` attribute, and the exact XML element or attribute name to be used can be supplied using an ```[XmlElement]``` or ```[XmlAttribute]``` attribute. Note that the second property must fully support reading and writing to work property with ```XmlSerializer```.

2. For more complicated scenarios, there is an interface that ```XmlSerializer``` checks for. Data types that implement ```IXmlSerializable``` can take complete charge over the translation of XML data to/from C# objects.

More information can be found at MSDN ([here](http://msdn.microsoft.com/en-us/library/2baksw0z(v=vs.110).aspx) and [here](http://msdn.microsoft.com/en-us/library/system.xml.serialization.ixmlserializable(v=vs.110).aspx)), as well as in many tutorials available on the web ([here's one](http://www.codeproject.com/Articles/43237/How-to-Implement-IXmlSerializable-Correctly)).

[<< Part 20. Content negotiation](Content negotiation) - [Documentation overview](Documentation) - [Part 22. Authentication >>](Authentication overview)