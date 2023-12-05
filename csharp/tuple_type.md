[C#] Tuple 類型

參考自 MSDN [Tuple types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)

想在函式傳回多個回傳值在 .Net 4.0 之前有 2 種方式：
- 宣告一個 Class，將所有佪傳值放進此 Class
- 在參數前面使用 out 或 ref 關鍵字

建立 Class 可能遇到幾個問題
- 只有此函式用到此 Class，你會建立許多一次性的 Class，造成閱讀困難
- 有些回傳值不適合放在同一個 Class 裡，例如當一個 Class 被取名叫 BookInfo，裡面不太適合放入人名的資訊

使用 out 或 ref 可能遇到幾個問題
- 回傳值太多，造成函式的參數數目太多，不易閱讀

.Net 4.0 之後提供另一種選擇， Tuple 類型，是 Value Type，如下圖用 Item1, Item2 取回傳值

```csharp
public static (string, int) GetInfo()
{
    return ("David", 40);
}
```

```csharp
var result = GetInfo();
Console.WriteLine("Name: {0}, Age: {1}", result.Item1, result.Item2);
```
>Name: David, Age: 40


Tuple 也可以跟 ref 關鍵字一起用

```csharp
public static void GetInfo(ref (string, int) input)
{
    input = ("David", 40);
}
```

```csharp
(string, int ) result = ("AAA", 3);
Console.WriteLine("Name: {0}, Age: {1}", result.Item1, result.Item2);

GetInfo(ref result);
Console.WriteLine("Name: {0}, Age: {1}", result.Item1, result.Item2);
```
>Name: AAA, Age: 3  
Name: David, Age: 40

Tuple 也可以跟 out  關鍵字一起用

```csharp
ar limitsLookup = new Dictionary<int, (int Min, int Max)>()
{
    [2] = (4, 10),
    [4] = (10, 20),
    [6] = (0, 23)
};

if (limitsLookup.TryGetValue(4, out (int Min, int Max) limits))
{
    Console.WriteLine($"Found limits: min is {limits.Min}, max is {limits.Max}");
}
```
>Found limits: min is 10, max is 20

使用上仍要注意，回傳值只能叫 Item1, Item2 ⋯⋯ 可讀性變差，最好
- 控制回傳值數量
- 在函式加入 summary 注解
- 不要跨太多層函式呼叫

.Net 7.0 則更進一步增加可讀性，可以指定參數名稱

```csharp
public static (string Name, int Age) GetInfo()
{
    return ("David", 40);
}
```

```csharp
var result = GetInfo();
Console.WriteLine("Name: {0}, Age: {1}", result.Name, result.Age);
```
>Name: David, Age: 40

C# Tuple 是由類型所支援 System.ValueTuple ，與以類型表示的 System.Tuple Tuple 不同。 主要差異如下：
- System.ValueTuple 類型是 實值型別 。 System.Tuple 類型是 參考型別 。
- System.ValueTuple 類型是可變的。 System.Tuple 類型是不可變的。
- 類型的資料成員 System.ValueTuple 是欄位。 類型的資料成員 System.Tuple 是屬性。