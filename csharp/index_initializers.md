# [C# 6.0]Index Initializer

C# 6.0 以前要初始化集合，需要這樣寫
```csharp
Dictionary<string, int> dict = new Dictionary<string, int>()
{
    { "index1", 1 },
    { "index12", 2 }
};
```

<br/>C# 6.0 以後可以這樣寫
```csharp
Dictionary<string, int> dict = new Dictionary<string, int>()
{
    ["index1"] =  1,
    ["index2"] =  2,
};
```

## 相同的 key
舊的寫法不允許有相同的 key，
```csharp
// compiler error
Dictionary<string, int> dict = new Dictionary<string, int>()
{
    { "index1", 1 },
    { "index11", 99 }
};
```
>Unhandled exception. System.ArgumentException: An item with the same key has already been added. Key: index1

<br/>新寫法可以允許相同的 key，就是後蓋前
```csharp
Dictionary<string, int> dict = new Dictionary<string, int>()
{
    ["index1"] =  1,
    ["index1"] =  99,
};
```

## 其它例子

```csharp
public class MyClass
{
    private int[] _intArray = new int [10];
    private Dictionary<string, string> _dict = new Dictionary<string, string>();

    public int this[int index]
    {
        get => _intArray[index];
        set => _intArray[index] = value;
    }
    
    public string this[string index]
    {
        get => _dict[index];
        set => _dict[index] = value;
    }
}
```

```csharp
var myClass = new MyClass()
{
    [1] = 10,
    [2] = 20,
    ["Hello"] = "World",
    ["Microsoft"] = "Teams",
};
```