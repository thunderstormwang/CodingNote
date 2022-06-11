# [C# 6.0]nameof Operator

直接指派變數名稱，不用複製貼上，且變數更名後也會跟著更名

範例一
```csharp
var Foo = "Hello World";

Console.WriteLinke($"valne in {nameof(Foo)} is {Foo}")
```
>valne in Foo is Hello World

<br/>範例二
```csharp
public void MyFunction(string greedIsGood)
{
    if (string.IsNullOrEmpty(greedIsGood))
    {
        throw new ArgumentNullException($"{nameof(MyFunction)}: {greedIsGood}");
    }
}
```