# foreach 迴圈 和 Linq 的 ForEach 的差別

- foreach 迴圈是基本的語言結構，而 Linq 的 ForEach 是擴充方法
- foreach 在逐一巡覽元素時，不能改變元素，而 ForEach 可以
  - IEnumerable\<T> 在 foreach 可以直接巡覽，但需要先將它轉成 List\<T> 才能呼叫 ForEach

在 foreach 會 build error
```csharp
var names = new List<string> { "Alice", "Bob", "Charlie", "David" };
foreach (var name in names)
{
    name = name.ToUpper();
    Console.WriteLine(name);
}
```

在 ForEach 可以正常運行
```csharp
var names = new List<string> { "Alice", "Bob", "Charlie", "David" };
names.ForEach(name =>
{
    name = name.ToUpper();
    Console.WriteLine(name);
});
```

要注意的點是，ForEach 方法是是回傳 void，它無法做非同步的等待  

這會逐一執行，並等待 5 秒後再換下個元素
```csharp
foreach (var name in names)
{
    Console.WriteLine($"Processing {name}");
    await Task.Delay(5000);
    Console.WriteLine($"Processed {name}");
}
```

這會將所 Task 都執行，但不會等待其結果
```csharp
names.ForEach(async name =>
{
    Console.WriteLine($"Processing {name}");
    await Task.Delay(5000);
    Console.WriteLine($"Processed {name}");
});
```

除非將上述改成
```csharp
var tasks = names.ForEach(async name =>
{
    Console.WriteLine($"Processing {name}");
    await Task.Delay(5000);
    Console.WriteLine($"Processed {name}");
});
await Task.WhenAll(tasks)
```

---
參考自
- [LINQ ForEach() 與 async / await](https://blog.darkthread.net/blog/linq-foreach-n-async/?fbclid=IwAR2YXxHg-Wk5Lcr-XRHiPUKMg1nYmZwwLxepO-3OhOb7gkbBQlOcpIag-e8)
- [foreach vs ForEach](https://ericlippert.com/2009/05/18/foreach-vs-foreach/)