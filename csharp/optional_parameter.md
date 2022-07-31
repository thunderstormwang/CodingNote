# [C# 4.0]Optional Parameter 選擇性參數

在函式宣告時，加上預設值
```csharp
public void MyTestFunc(double p1, int p2 = 100, string p3 = "HelloWorld")
{
    Console.WriteLinke($"p1: {0:0.00}, p2: {1}, p3: {2}", p1, p2, p3);
}
```

沒給值的參數，使用預設值，減少使用函式重載

<br/>皆給予值
```csharp
MyTestFunc(3.3, 40, "Assain's Creed");
```
>p1: 3.30, p2: 40, p3:  "Assain's Creed"

<br/>p3 使用選擇性參數
```csharp
MyTestFunc(3.3, 40);
```
>p1: 3.30, p2: 40, p3:  "HelloWorld"

<br/>p2, p3 使用選擇性參數
```csharp
MyTestFunc(3.3);
```
>p1: 3.30, p2: 100, p3:  "HelloWorld"

<br/>無法跳過中間的參數，必須從該參數之後都使用選擇性參數
```csharp
// compiler error
MyTestFunc(2000, , "Assain's Creed");
```

<br/>可以搭配具名參數跳過中間的參數
```csharp
MyTestFunc(p3: "Assain's Creed", p1: 2000 );
```