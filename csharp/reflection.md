# [C#]反射 Reflection

記錄用過的寫法

- [[C#]反射 Reflection](#c反射-reflection)
  - [todo: attribute](#todo-attribute)
  - [取得所有 public fields](#取得所有-public-fields)
  - [取得特定 private field](#取得特定-private-field)
  - [設定特定 private field](#設定特定-private-field)
  - [取得所有 public properties](#取得所有-public-properties)
  - [取得特定 private property](#取得特定-private-property)
  - [設定特定 private property](#設定特定-private-property)
  - [呼叫 Private Method](#呼叫-private-method)
  - [取得特定 attribute](#取得特定-attribute)
  - [取得所有掛有指定 attribute 的 property](#取得所有掛有指定-attribute-的-property)

todo: attribute
---
## 取得所有 public fields
宣告類別
```csharp
public class MyData
{
    public int fieldInt = 10;
    public double fieldDouble = 123.12d;
    private string privateFieldString = "Hello World!!";
}
```
<br/>取得所有 public fields 的方法
```csharp
public void LoopThroughField(object data)
{
    foreach (var fieldInfo in data.GetType().GetFields())
    {
        var name = fieldInfo.Name;
        var temp = fieldInfo.GetValue(data);
        if (temp is int)
        {
            var value = (int)temp;
            Console.WriteLine($"{name} is int, value: {value}");
        }
        else if (temp is double)
        {
            var value = (double)temp;
            Console.WriteLine($"{name} is double, value: {value}");
        }
        else if (temp is string)
        {
            var value = (string)temp;
            Console.WriteLine($"{name} is string, value: {value}");
        }
    }
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
LoopThroughField(myData);
```
>fieldInt is int, value: 10
<br/>fieldDouble is double, value: 123.12

---
## 取得特定 private field
取得特定 private field 的方法
```csharp
public void GetField(object data, string fieldName)
{
    var fieldInfo = data.GetType().GetFields(BindingFlags.NonPublic | BindingFlags.Instance)
        .FirstOrDefault(f => f.Name == fieldName);
    if (fieldInfo == null)
    {
        return;
    }

    var name = fieldInfo.Name;
    var temp = fieldInfo.GetValue(data);
    if (temp is not string)
    {
        return;
    }

    var value = (string)temp;
    Console.WriteLine($"{name} is int, value: {value}");
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
GetField(myData, "privateFieldString");
```
>privateFieldString is int, value: Hello World!!

---
## 設定特定 private field
取得特定 private field 的方法
```csharp
public void SetField(object data, string fieldName, object newValue)
{
    var fieldInfo = data.GetType().GetFields(BindingFlags.NonPublic | BindingFlags.Instance)
        .FirstOrDefault(f => f.Name == fieldName);
    if (fieldInfo == null)
    {
        return;
    }

    var name = fieldInfo.Name;
    fieldInfo.SetValue(data, newValue);
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
SetField(myData, "privateFieldString", "ABCDEFG");
```

---
## 取得所有 public properties
宣告類別
```csharp
public class MyData
{
    public int propertyInt { get; set; }
    public double PropertyDouble { get; set; }
    private string privatePropertyString { get; set; }

    public MyData()
    {
        propertyInt = 10;
        PropertyDouble = 123.12d;
        privatePropertyString = "Hello World!!";
    }
}
```
<br/>取得所有 public properties 的方法
```csharp
public void LoopThroughProperty(object data)
{
    foreach (var fieldInfo in data.GetType().GetProperties())
    {
        var name = fieldInfo.Name;
        var temp = fieldInfo.GetValue(data);
        if (temp is int)
        {
            var value = (int)temp;
            Console.WriteLine($"{name} is int, value: {value}");
        }
        else if (temp is double)
        {
            var value = (double)temp;
            Console.WriteLine($"{name} is double, value: {value}");
        }
        else if (temp is string)
        {
            var value = (string)temp;
            Console.WriteLine($"{name} is string, value: {value}");
        }
    }
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
LoopThroughProperty(myData);
```
>propertyInt is int, value: 10
<br/>PropertyDouble is double, value: 123.12

---
## 取得特定 private property
取得特定 private property 的方法
```csharp
public void GetProperty(object data, string fieldName)
{
    var fieldInfo = data.GetType().GetProperties(BindingFlags.NonPublic | BindingFlags.Instance)
        .FirstOrDefault(f => f.Name == fieldName);
    if (fieldInfo == null)
    {
        return;
    }

    var name = fieldInfo.Name;
    var temp = fieldInfo.GetValue(data);
    if (temp is not string)
    {
        return;
    }

    var value = (string)temp;
    Console.WriteLine($"{name} is int, value: {value}");
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
LoopThroughField(myData);
```
>privatePropertyString is int, value: Hello World!!

---
## 設定特定 private property
取得特定 private property 的方法
```csharp
public void SetProperty(object data, string fieldName, object newValue)
{
    var fieldInfo = data.GetType().GetProperties(BindingFlags.NonPublic | BindingFlags.Instance)
        .FirstOrDefault(f => f.Name == fieldName);
    if (fieldInfo == null)
    {
        return;
    }

    var name = fieldInfo.Name;
    fieldInfo.SetValue(data, newValue);
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
SetField(myData, "privateFieldString", "ABCDEFG");
```

---
## 呼叫 Private Method
宣告類別
```csharp
public class MyData
{
    private string Hello(string name)
    {
        return $"Hello, {name}";
    }
}
```
<br/>呼叫 Private Method 的方法
```csharp
private object ExecuteMethod(object instance, string methodName, object[] methodParam)
{
    var methodInfo = instance.GetType().GetMethod(methodName, BindingFlags.NonPublic | BindingFlags.Instance);

    if (methodInfo == null)
    {
        return null;
    }

    var result = methodInfo.Invoke(instance, methodParam);

    return result;
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
var result = ExecuteMethod(myData, "Hello", new object[] { "John" });
```

## 取得特定 attribute
宣告 enum
```csharp
public enum LightSwitch
{
    [Description("打開")]
    On = 1,

    [Description("關閉")]
    Off = 2,
}
```
<br/>宣告取得特定 attribute 的擴充方法
```csharp
public static class EnumExtension
{
    public static string GetDescription(this Enum @enum)
    {
        var atr = GetAttribute<DescriptionAttribute>(@enum);
        return atr != null ? atr.Description : string.Empty;
    }

    private static T GetAttribute<T>(Enum @enum) where T : Attribute
    {
        var fieldInfo = @enum.GetType().GetField(@enum.ToString());
        if (fieldInfo == null)
        {
            return null;
        }

        var attribute = fieldInfo.GetCustomAttribute(typeof(T), false) as T;
        return attribute;
    }
}
```
<br/>呼叫該方法
```csharp
Console.WriteLine($"LightSwitch.On: {LightSwitch.On.GetDescription()}");
Console.WriteLine($"LightSwitch.Off: {LightSwitch.Off.GetDescription()}");
```

>LightSwitch.On: 打開
<br/>LightSwitch.Off: 關閉

## 取得所有掛有指定 attribute 的 property

宣告自訂 attribute
```csharp
public class PrintAttribute : Attribute
{}
```
<br/>宣告類別，並在其中一個 property 掛上 PrintAttribute
```csharp
public class MyData
{
    public int propertyInt { get; set; }
    
    [PrintAttribute]
    public double PropertyDouble { get; set; }
    
    private string privatePropertyString { get; set; }

    public MyData()
    {
        propertyInt = 10;
        PropertyDouble = 123.12d;
        privatePropertyString = "Hello World!!";
    }
}
```
<br/>取得所有掛有指定 attribute 的 property 的方法
```csharp
public void LoopThroughProperty(object data, Type attrType)
{
    foreach (var fieldInfo in data.GetType().GetProperties().Where( prop => prop.IsDefined(attrType, false)))
    {
        var name = fieldInfo.Name;
        var temp = fieldInfo.GetValue(data);
        if (temp is int)
        {
            var value = (int)temp;
            Console.WriteLine($"{name} is int and have PrintAttribute, value: {value}");
        }
        else if (temp is double)
        {
            var value = (double)temp;
            Console.WriteLine($"{name} is double and have PrintAttribute, value: {value}");
        }
        else if (temp is string)
        {
            var value = (string)temp;
            Console.WriteLine($"{name} is string and have PrintAttribute, value: {value}");
        }
    }
}
```
<br/>呼叫該方法
```csharp
var myData = new MyData();
LoopThroughProperty(myData, typeof(PrintAttribute));
```
>PropertyDouble is double and have PrintAttribute, value: 123.12