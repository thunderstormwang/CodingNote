# [C# 3.0] Linq deferred execution - Linq 延遲執行

## 等到要用時才會評估運算式
為了方便看執行順序，宣告以下函式
```csharp
public bool MyWhere(int input)
{
    Console.WriteLine($"MyWhere, input: {input}");
    return input >= 2;
} 
```
```csharp
public int MySelect(int input)
{
    Console.WriteLine($"MySelect, input: {input}");
    return input;
} 
```
```csharp
var numbers = new List<int>() { 1, 2, 3 };

var result = numbers.Where(n => MyWhere(n)).Select(n => MySelect(n));

Console.WriteLine("i am first");

foreach (var item in result)
{
    Console.WriteLine($"item: {item}");
}
```
>i am first
<br/>MyWhere, input: 1
<br/>MyWhere, input: 2
<br/>MySelect, input: 2
<br/>item: 2
<br/>MyWhere, input: 3
<br/>MySelect, input: 3
<br/>item: 3

<br/>由結果可以看到，等到真的要評估運算式例如呼叫 foreach 時，才會去呼叫MyWhere, MySelect


---
## 每次都會執行運算式

```csharp
var numbers = new List<int>() { 1, 2, 3 };

var result = numbers.Where(n => MyWhere(n)).Select(n => MySelect(n));

foreach (var item in result)
{
    Console.WriteLine($"item: {item}");
}

Console.WriteLine("======");

foreach (var item in result)
{
    Console.WriteLine($"item: {item}");
}
```
>MyWhere, input: 1
<br/>before foreach
<br/>MyWhere, input: 2
<br/>MySelect, input: 2
<br/>item: 2
<br/>MyWhere, input: 3
<br/>MySelect, input: 3
<br/>item: 3
<br/>======
<br/>before foreach
<br/>MyWhere, input: 1
<br/>MyWhere, input: 2
<br/>MySelect, input: 2
<br/>item: 2
<br/>MyWhere, input: 3
<br/>MySelect, input: 3
<br/>item: 3

<br/>由結果可以看到，每次在評估運算式例如呼叫 foreach 時，都會去呼叫MyWhere, MySelect

---
## 篩選的條件固定後，應該要評估運算式
當篩選的條件固定後，評估運算式，例如呼叫 ToList()，避免重覆的運算
```csharp
var result = numbers.Where(n => MyWhere(n)).Select(n => MySelect(n)).ToList()
```