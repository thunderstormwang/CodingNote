# [C# 4.0]Named Parameter 具名參數

宣告函式
```csharp
public void MyTestFunc(double p1, int p2, string p3, string p4)
{
    Console.WriteLinke($"p1: {p1:0.00}, p2: {p2}, p3: {p3}, p4: {p4}");
}
```
可以用以下方法呼叫

<br/>正常的呼叫方法, 照順序填入參數
```csharp
MyTestFunc(33.3, 500, "Ubisoft", "Assassin's Creed");
```
使用具名參數, 可不照順序
```csharp
MyTestFunc(p2: 500, p3: "Ubisoft", p4: "Assassin's Creed", p1: 33.3);
```
前面照順序, 後面用具名參數且不照順序
```csharp
MyTestFunc(33.3, p3: "Ubisoft", p2: 500, p4: "Assassin's Creed");
```
以上結果都是
>p1: 33.3, p2: 500, p3: Ubisoft, p4: Assassin's Creed

<br/>一旦使用具名參數後, 後面都須要比照具名參數的方式給值
<br/>p1 照順序, p2, p3 用具名參數, 那麼排在後面的 p4 也需要用具名參數
```csharp
MyTestFunc(33.3, p3: "Ubisoft", p2: 500, "Assassin's Creed");
```
compiler 會提示編繹錯誤:
>Named argument specifications must appear after all fixed arguments have been specified