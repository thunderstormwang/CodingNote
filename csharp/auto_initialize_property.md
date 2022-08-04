# [C# 6.0]Auto-Initialize Property - 初始化自動實作的屬性

C# 6.0 以前，針對只有 get 的 property，只能在建構子做初始化。
```csharp
public class Coordinate
{
    public int X { get; }

    public Coordinate()
    {
        X = 33;
    }
}
```

<br/>C# 6.0 後可以這樣對 Auto Property 作初始化
```csharp
public class Coordinate
{
    public int X { get; set; } = 33;

    public int Y { get; } = 60;
}
```

<br/>也可以使用 static 函式
```csharp
public class Coordinate
{
    public int X { get; set; } = Num();

    private static int Num()
    {
        return 33;
    }
}
```