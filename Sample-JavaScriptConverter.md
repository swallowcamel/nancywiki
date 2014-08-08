```c#
using Nancy.Json;

public class DatePartsConverter : JavaScriptConverter
{
  public override IEnumerable<Type> SupportedTypes
  {
    get { yield return typeof(DateTime); }
  }

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
}
```