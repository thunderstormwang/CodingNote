# [C# 6.0]Expression-bodied members - 運算式主題成員

在 C# 6.0，你可以在 property, method, indexer 用 lambda 作宣告

## Property
```csharp
public class Location
{
   private string locationName;

   public string Name
   {
      get => locationName;
      set => locationName = value;
   }
}
```
等同以下寫法
```csharp
public class Location
{
   private string locationName;
   public string Name
   {
      get { return locationName; }
      set { locationName = value; }
   }
}
```

## method
```csharp
public Multiple(int a, int b)
{
    return a * b;
}
```
等同以下寫法
```csharp
public Multiple(int a, int b) => a * b;
```

## indexer
```csharp
public class Grade
{
    private Dictionary<string, int> _scoreDict = new Dictionary<string, int>();

    public int this[string name] => _scoreDict[name];
}
```
等同以下寫法
```csharp
public class Grade
{
    private Dictionary<string, int> _scoreDict = new Dictionary<string, int>();

    public int this[string name]
    {
        get
        {
            return _scoreDict[name];
        }
    }
}
```

