# [C# 6.0]Exception Filter
在 catch 區塊可用 when 關鍵字指定要不要處理此例外
```csharp
var a = 7;
var b = 0;

try
{
    var result = a / b;
}
catch (Exception ex) when (b != 0)
{
    Console.WriteLine($"你看不到這行, {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"我會被執行, {ex.Message}");
}
```
>我會被執行, Attempted to divide by zero.