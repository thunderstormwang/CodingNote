# [C# 3.0]var type - var 型態

用 var 關鍵字，告訴 compiler 在 compile time 幫你決定該變數的型別，可用內建型別、匿名類別、自訂類別。
<br/>如果 compiler 無法推斷其型別就會跳 compile error。
<br/>通常用於LINQ，用 var 讓 compiler 幫你決定取出的型別，如此只要 LINQ 的委派的型別一改，接回傳值的型別不用跟著改。

```csharp
var number = 5;
Console.WriteLine(number.GetType().Name)

var array = new [] { 33, 44, 55};
Console.WriteLine(array.GetType().Name)
```
>Int32
<br/>Int32[]

<br/>沒給初始值，compiler 無法推斷型別
```csharp
// compiler error
var notAssigned;
```

<br/>compiler 找不到合適的型別
```csharp
// compiler error
var multiType = new [] { 33, "44", 55.5 };
```

<br/>一旦宣告後，不能指派另一型別
```csharp
// compiler error
var reassigned = 5;
reassigned = "Hello World";
```


<br/>支援繼承
```csharp
public class Parent
{}

public class Child : Parent
{}
```
```csharp
var reassigned = new Parent();
reassigned = new Child();
```



