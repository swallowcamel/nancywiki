```c#
using Nancy.Json;

public class ByteArrayAsBase64Converter : JavaScriptPrimitiveConverter
{
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
}
```