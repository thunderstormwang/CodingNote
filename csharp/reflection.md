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
```csharp
var myData = new MyData();
LoopThroughField(myData);
```
<br/>取得所有 public fields
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
            Console.WriteLine($"{name} is int, value: {value}");
        }
        else if (temp is string)
        {
            var value = (string)temp;
            Console.WriteLine($"{name} is string, value: {value}");
        }
    }
}
```
>fieldInt is int, value: 10
<br/>fieldDouble is int, value: 123.12

---
## 取得特定 private field
```csharp
var myData = new MyData();
GetField(myData, "privateFieldString");
```
<br/>取得特定 private field
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
>privateFieldString is int, value: Hello World!!

---
## 設定特定 private field

```csharp
var myData = new MyData();
SetField(myData, "privateFieldString", "ABCDEFG");
```
<br/>取得特定 private field
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
```csharp
var myData = new MyData();
LoopThroughProperty(myData);
```
<br/>取得所有 public properties
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
            Console.WriteLine($"{name} is int, value: {value}");
        }
        else if (temp is string)
        {
            var value = (string)temp;
            Console.WriteLine($"{name} is string, value: {value}");
        }
    }
}
```
>propertyInt is int, value: 10
<br/>PropertyDouble is int, value: 123.12

---
## 取得特定 private property
```csharp
var myData = new MyData();
LoopThroughField(myData);
```
<br/>取得特定 private property
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
>privatePropertyString is int, value: Hello World!!

---
## 設定特定 private property

```csharp
var myData = new MyData();
SetField(myData, "privateFieldString", "ABCDEFG");
```
<br/>取得特定 private property
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

---
## 呼叫 Private Method

```csharp
public class MyData
{
    private string Hello(string name)
    {
        return $"Hello, {name}";
    }
}
```
```csharp
var myData = new MyData();
var result = ExecuteMethod(myData, "Hello", new object[] { "John" });
```
<br/>呼叫 Private Method
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