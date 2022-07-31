# [C# 3.0]Auto Property 自動實作的屬性

C# 3.0 後，這樣寫
```csharp
public class Game
{
    public string Title { get; set; }
}
```

<br/>compiler 會幫你轉下成列語法
```csharp
public class Game
{
    public string _title { get; set; }
    public string Title
    {
        get { return titile; }
        set { _titile = value; }
    }
}
```