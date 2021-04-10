# c-sharp-common-extension-method
C# Common Extension Methods

https://github.com/rohit-patel/c-sharp-common-extension-method

```
public static class CommonExtensions
{
    public static string ToHHMMSSString(this int seconds)
    {
        TimeSpan span = TimeSpan.FromSeconds(seconds);
        if (span != null)
        {
            int hours = (span.Days * 24) + span.Hours;
            return string.Format("{0}:{1}:{2}", hours.ToString("00"), span.Minutes.ToString("00"), span.Seconds.ToString("00"));
        }
        else
            return "00:00:00";
    }

    public static string ToHHMMSSString(this int? seconds)
    {
        return seconds.HasValue ? seconds.Value.ToHHMMSSString() : string.Empty;
    }

    public static string ToLocalTime(this DateTime date, string timeZoneName)
    {
        return string.Format("{0}", LocalTime(date, timeZoneName).ToString("MM/dd/yyyy hh:mm tt"));
    }
    public static string ToLocalTime(this DateTime? date, string timeZoneName)
    {
        return date.HasValue ? date.Value.ToLocalTime(timeZoneName) : string.Empty;
    }

    public static DateTime LocalTime(DateTime dateTimeToConvert, string timeZone)
    {
        if (string.IsNullOrWhiteSpace(timeZone))
            timeZone = TimeZone.CurrentTimeZone.StandardName;

        TimeZoneInfo destinationTimeZone;
        try
        {
            destinationTimeZone = TimeZoneInfo.FindSystemTimeZoneById(timeZone);
        }
        catch (Exception)
        {
            destinationTimeZone = TimeZoneInfo.FindSystemTimeZoneById(TimeZone.CurrentTimeZone.StandardName);
        }

        TimeZoneInfo sourceTimeZone = TimeZoneInfo.FindSystemTimeZoneById("UTC");

        return TimeZoneInfo.ConvertTime(dateTimeToConvert, sourceTimeZone, destinationTimeZone);
    }


    public static T ConvertToDataType<T>(object value)
    {
        return value != DBNull.Value ? (T)Convert.ChangeType(value, typeof(T)) : default(T);
    }


    /// <summary>
    /// Try to parse a string and convert it to a DateTime object. In
    /// case of an exception, the defaultValue will be returned.
    /// </summary>
    /// <param name="text"></param>
    /// <param name="defaultValue"></param>
    /// <returns></returns>
    public static DateTime ParseDateTime(string text, DateTime defaultValue)
    {
        DateTime result;
        return DateTime.TryParse(text, out result) ? result : defaultValue;
    }

    /// <summary>
    /// Try to parse a string and convert it to a DateTime object, or return
    /// the MinDate value in case of an error.
    /// </summary>
    /// <param name="text"></param>
    /// <returns></returns>
    public static DateTime ParseDateTime(this string text)
    {
        return ParseDateTime(text, DateTime.MinValue);
    }

    public static DataTable ToDataTable<T>(this List<T> items)
    {
        PropertyDescriptorCollection properties = TypeDescriptor.GetProperties(typeof(T));
        DataTable dataTable = new DataTable();
        foreach (PropertyDescriptor prop in properties)
            dataTable.Columns.Add(prop.Name, Nullable.GetUnderlyingType(prop.PropertyType) ?? prop.PropertyType);
        foreach (T item in items)
        {
            DataRow row = dataTable.NewRow();
            foreach (PropertyDescriptor prop in properties)
                row[prop.Name] = prop.GetValue(item) ?? DBNull.Value;
            dataTable.Rows.Add(row);
        }
        return dataTable;
    }

    public static bool Between(this int num, int lower, int upper, bool inclusive = false)
    {
        return inclusive
            ? lower <= num && num <= upper
            : lower < num && num < upper;
    }

    public static decimal ToRound(this decimal value)
    {
        return Math.Round(value, 2, MidpointRounding.AwayFromZero);
    }
    public static decimal? ToRound(this decimal? value)
    {
        return value.HasValue ? value.Value.ToRound() : value;
    }

    public static double ToRound(this double value)
    {
        return Math.Round(value, 2, MidpointRounding.AwayFromZero);
    }
    public static double? ToRound(this double? value)
    {
        return value.HasValue ? value.Value.ToRound() : value;
    }

    public static String BytesToString(this long byteCount)
    {
        string[] suf = { "B", "KB", "MB", "GB", "TB", "PB", "EB" }; //Longs run out around EB
        if (byteCount == 0)
            return "0" + suf[0];
        long bytes = Math.Abs(byteCount);
        int place = Convert.ToInt32(Math.Floor(Math.Log(bytes, 1024)));
        double num = Math.Round(bytes / Math.Pow(1024, place), 1);
        return (Math.Sign(byteCount) * num).ToString() + " " + suf[place];
    }

    public static object ToAnonymousObject(this IDictionary<string, object> @this)
    {
        var expandoObject = new ExpandoObject();
        var expandoDictionary = (IDictionary<string, object>)expandoObject;

        foreach (var keyValuePair in @this)
        {
            expandoDictionary.Add(keyValuePair);
        }
        return expandoObject;
    }
    public static T GetValueFromDataRow<T>(this DataRow row, string columnName)
    {
        if (row.Table.Columns.Contains(columnName) && (object)row[columnName] != DBNull.Value)
        {
            return (T)row[columnName];
        }
        return default(T);
    }

    public static T ConvertTo<T>(object value)
    {
        return value != DBNull.Value ? (T)Convert.ChangeType(value, typeof(T)) : default(T);
    }
    public static byte[] ReadFully(this Stream input)
    {
        using (MemoryStream ms = new MemoryStream())
        {
            input.CopyTo(ms);
            return ms.ToArray();
        }
    }

    public static string ToInitilizeCharacters(this string source, int length, bool isUperCase = true)
    {
        source = (source ?? String.Empty);
        if (source.Length >= length)
        {
            if (isUperCase)
                source = source.Substring(0, length).ToUpper();
            else
                source = source.Substring(0, length);
        }
        return source;
    }

    public static double ToUnixTimestamp(this DateTime dateTime)
    {
        DateTime origin = new DateTime(1970, 1, 1, 0, 0, 0, 0);
        TimeSpan diff = dateTime - origin;
        return Math.Floor(diff.TotalSeconds);
    }

    public static DateTime FromUnixTimestamp(this double unixMS)
    {
        //  unixMS = Math.Floor(unixMS / 1000);
        System.DateTime dtDateTime = new DateTime(1970, 1, 1, 0, 0, 0, 0, System.DateTimeKind.Utc);
        dtDateTime = dtDateTime.AddSeconds(unixMS);
        return dtDateTime;
    }

    // convert to tenant time


    // convert to utc time
    public static DateTime ToUtcDateTime(this DateTime dateTimeToConvert, string tenantTimeZone)
    {
        if (string.IsNullOrWhiteSpace(tenantTimeZone))
            tenantTimeZone = TimeZone.CurrentTimeZone.StandardName;
        TimeZoneInfo easternZone = TimeZoneInfo.FindSystemTimeZoneById(tenantTimeZone);
        return TimeZoneInfo.ConvertTimeToUtc(dateTimeToConvert, easternZone);
    }

    // convert to utc kind
    public static DateTime ToUtcKind(this DateTime dateTimeToConvert)
    {
        return DateTime.SpecifyKind(dateTimeToConvert, DateTimeKind.Utc);
    }
    public static DateTime? ToUtcKind(this DateTime? dateTimeToConvert)
    {
        if (dateTimeToConvert.HasValue)
            return ToUtcKind(dateTimeToConvert.Value);
        else
            return null;
    }

    /// <summary>
    /// Get a substring of the first N characters.
    /// </summary>
    public static string Truncate(this string source, int length, string appenString = "")
    {
        source = (source ?? String.Empty);
        if (source.Length > length)
        {
            source = source.Substring(0, length) + appenString;
        }
        return source;
    }



    public static List<dynamic> ToDynamic(this DataTable dt)
    {
        var dynamicDt = new List<dynamic>();
        foreach (DataRow row in dt.Rows)
        {
            dynamic dyn = new ExpandoObject();
            dynamicDt.Add(dyn);
            foreach (DataColumn column in dt.Columns)
            {
                var dic = (IDictionary<string, object>)dyn;
                dic[column.ColumnName] = row[column];
            }
        }
        return dynamicDt;
    }

    public static T GetValue<T>(this JObject obj, string fieldName)
    {
        if (obj != null && obj.ContainsKey(fieldName))
        {
            JToken token = obj[fieldName];
            return token.Value<T>();
        }
        return default(T);
    }
}

public static class EnumHelper
{
    // This extension method is broken out so you can use a similar pattern with 
    // other MetaData elements in the future. This is your base method for each.
    //In short this is generic method to get any type of attribute.
    public static T GetAttribute<T>(this Enum value) where T : Attribute
    {
        var type = value.GetType();
        var memberInfo = type.GetMember(value.ToString());
        var attributes = memberInfo[0].GetCustomAttributes(typeof(T), false);
        return (T)attributes.FirstOrDefault();//attributes.Length > 0 ? (T)attributes[0] : null;
    }

    // This method creates a specific call to the above method, requesting the
    // Display MetaData attribute.
    //e.g. [Display(Name = "Sunday")]
    public static string ToDisplayName(this Enum value)
    {
        var attribute = value.GetAttribute<DisplayAttribute>();
        return attribute == null ? value.ToString() : attribute.Name;
    }


    // This method creates a specific call to the above method, requesting the
    // Description MetaData attribute.
    //e.g. [Description("Day of week. Sunday")]
    public static string ToDescription(this Enum value)
    {
        var attribute = value.GetAttribute<DescriptionAttribute>();
        return attribute == null ? value.ToString() : attribute.Description;
    }

}
```
