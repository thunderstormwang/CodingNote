# [C# 3.0] Linq deferred execution - Linq 延遲執行

## 等到要用時才會評估運算式

為了方便看執行順序，在 `Where`, `Select` 印出訊息，宣告以下函式

```csharp
var numbers = new List<int>() { 1, 2, 3 };

var result = numbers.Where(n => 
    {
        Console.WriteLine($"Where, input: {input}");
        return n >= 2;
    })
    .Select(n => 
    {
        Console.WriteLine($"Select, input: {input}");
        return n;
    });

Console.WriteLine("before foreach");

foreach (var item in result)
{
    Console.WriteLine($"foreach item: {item}");
}
```

>before foreach  
Where, input: 1  
Where, input: 2  
Select, input: 2  
foreach item: 2  
Where, input: 3  
Select, input: 3  
foreach item: 3  

<br/>由結果可以看到，等到真的要評估運算式例如呼叫 foreach 時，才會去呼叫MyWhere, MySelect  

---

## 每次都會執行運算式

```csharp
var numbers = new List<int>() { 1, 2, 3 };

var result = numbers.Where(n => 
    {
        Console.WriteLine($"Where, input: {input}");
        return n >= 2;
    })
    .Select(n => 
    {
        Console.WriteLine($"Select, input: {input}");
        return n;
    });

Console.WriteLine("before foreach 1");    

foreach (var item in result)
{
    Console.WriteLine($"item: {item}");
}

Console.WriteLine("before foreach 2");

foreach (var item in result)
{
    Console.WriteLine($"item: {item}");
}
```
>before foreach 1  
Where, input: 1  
Where, input: 2  
Select, input: 2  
foreach item: 2    
Where, input: 3  
Select, input: 3  
foreach item: 3  
before foreach 2  
Where, input: 1    
Where, input: 2  
Select, input: 2  
foreach item: 2    
Where, input: 3  
Select, input: 3  
foreach item: 3  

由結果可以看到，每次在評估運算式例如呼叫 foreach 時，都會去呼叫 `Where`, `Select`

---

## 篩選的條件固定後，應該要評估運算式

當篩選的條件固定後，評估運算式，例如呼叫 ToList()，避免重覆的運算  

```csharp
var result = numbers.Where(n => MyWhere(n)).Select(n => MySelect(n)).ToList()
```