# [C# 1.0]Delegate >> [C# 2.0]Anonymous Method >> [C# 3.0]Lambda

## lambda 的演進

<br/>宣告 delegate
```csharp
public delegate bool CompareDelegate(int x, int y);
```

<br/>宣告一個函式, 接受該 delegate 作為參數
```csharp
public bool MyTest(CompareDelegate func, int x, int y)
{
    return func(x, y);
}
```

### [C# 1.0]Delegate
在 C# 1.0 你得宣告符合該 delegate 的函式，也需要為該函式取名字，即使它很簡短
<br/>宣告符合該 delegate 的函式
```csharp
public bool MyCompare(int x, int y)
{
    return x > y;
}
```
<br/>然後可以這樣使用
```csharp
CompareDelegate comDel = MyCompare;
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate(MyCompare), 10, 5));
Console.WriteLine(MyTest(MyCompare, 10, 5));
```
>True
<br/>True
<br/>True

### [C# 2.0]Anonymous Method
C# 2.0 新增匿名函式，不用再為簡短的函式命名
<br/>使用時機是當這個 method 只會使用一次，且程式碼很少時。缺點是降低可讀性
```csharp
CompareDelegate comDel = delegate(int x, int y) { return x > y; };
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate(delegate(int x, int y) { return x > y; }), 
    10, 5));
Console.WriteLine(MyTest(delegate(int x, int y) { return x > y; }, 
    10, 5));
```
>True
<br/>True
<br/>True

### [C# 3.0]Lambda

到了 C# 3.0 又加入 Lamda 運算式，概念與匿名方法相似，但功能更為強大，而且也更簡潔。
<br/>(節錄自 MSDN：這兩種功能合稱為「匿名函式」(Anonymous Function)。一般而言，以 .NET Framework 3.5 版 (含) 以後版本做為目標的應用程式，都應該使用 Lambda 運算式。)

<br/>如何從匿名函式到 lambda
1. 把關鍵字 delegate 去掉
2. 加上 => 運算子, 這個運算子的左邊要放匿名方法的參數宣告, 右邊則是匿名方法的本體. => 的意思是 "goes to", 亦即「丟到」或「傳入」的意思

```csharp
CompareDelegate comDel = (int x, int y) =>
{
    return x > y;
};
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate((int x, int y) =>
    {
        return x > y;
    }),
    10, 5));
Console.WriteLine(MyTest((int x, int y) =>
    {
        return x > y;
    },
    10, 5));
```
>True
<br/>True
<br/>True

<br/>再進一步簡化 lambda
1. 大括孤和 return 都去掉
2. 程式碼排成一行
3. 去掉型別宣告, 讓 compiler 去推算
```csharp
CompareDelegate comDel = (x, y) => x > y;
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate((x, y) => x > y), 10, 5));
Console.WriteLine(MyTest((x, y) => x > y, 10, 5));
```
>True
<br/>True
<br/>True
---
## 以下為同樣的範例, 只是加上泛型

<br/>宣告 delegate
```csharp
public delegate bool CompareDelegate<T>(T x, T y);
```

<br/>宣告一個函式, 接受該 delegate 作為參數
```csharp
public bool MyTest(CompareDelegate<int> func, int x, int y)
{
    return func(x, y);
}
```

<br/>

### [C# 1.0]Delegate
<br/>宣告符合該 delegate 的函式
```csharp
public bool MyCompare(int x, int y)
{
    return x > y;
}
```
<br/>然後可以這樣使用
```csharp
CompareDelegate<int> comDel = MyCompare;
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate<int>(MyCompare), 10, 5));
Console.WriteLine(MyTest(MyCompare, 10, 5));
```
>True
<br/>True
<br/>True

<br/>

### [C# 2.0]Anonymous Method
```csharp
CompareDelegate<int> comDel = delegate(int x, int y) { return x > y; };
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate<int>(delegate(int x, int y) { return x > y; }), 
    10, 5));
Console.WriteLine(MyTest(delegate(int x, int y) { return x > y; }, 
    10, 5));
```
>True
<br/>True
<br/>True

<br/>

### [C# 3.0]Lambda

```csharp
CompareDelegate<int> comDel = (int x, int y) =>
{
    return x > y;
};
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate<int>((int x, int y) =>
    {
        return x > y;
    }),
    10, 5));
Console.WriteLine(MyTest((int x, int y) =>
    {
        return x > y;
    },
    10, 5));
```
>True
<br/>True
<br/>True

<br/>再進一步簡化 lambda
```csharp
CompareDelegate<int> comDel = (x, y) => x > y;
Console.WriteLine(MyTest(comDel, 10, 5));
Console.WriteLine(MyTest(new CompareDelegate<int>((x, y) => x > y), 10, 5));
Console.WriteLine(MyTest((x, y) => x > y, 10, 5));
```
>True
<br/>True
<br/>True
---
## lambda 注意事項

<br/>只有一個參數時, 可不用小括孤
```csharp
SingleParamDelegate sParam = x => x;
```

<br/>如果有多個參數, 就必須用一對小括孤將它們包住
```csharp
MultipleParamDelegate sParam = (x, y) => x + y;
```

<br/>如果沒有任何參數, 仍須有一對小括孤
```csharp
NoParamDelegate sParam = () => 5;
```

<br/>如果函式主體不只一行, 那就須要有大括孤和 return
```csharp
SingleParamDelegate sParam = () => 
{
    Console.WriteLinke("Hello World!");
    return 5;
};
```

<br/>如果委派方法中有 ref 或 out 參數, 就還是得明確宣告型別