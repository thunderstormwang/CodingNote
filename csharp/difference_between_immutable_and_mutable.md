# [C#]Immutable 和 Mutable 的差別

Mutable and immutable are English words that mean "can change" and "cannot change

Mutable
- it means "can cnahge" in Englihs.
- The mutable types are those whose data members can be changed after the instance is created.
- When we change the value of mutable objects, value is changed in same memory.

Immutable
- it means "cannot cnahge" in Englihs.
- Immutable types are those whose data members can not be changed after the instance is created.
- When we change the value of immutable objects, the new memory is created and the modified value is stored in new memory.

在 C# 通常都以 String, StringBuider 做為例子

String 是 Mutable。  
每次更新 String 變數的值，都會取得一塊新的記憶體存放新的字串，而舊的記憶體則等待 GC，所以頻繁地更動 string 變數會有效能問題。

```csharp
var str = string.Empty;

for (int i = 0; i < 10000; i++)   
{   
    str += "Modified ";   
}
```

StringBuilder 是 Immutable。  
每次更新 StringBuilder 變數的值，都是使用同一塊記憶體，沒有申請、收回記憶體體的效能問題
```csharp
var strB = new StringBuilder();   
   
for (int i = 0; i < 10000; i++)   
{   
    strB.Append("Modified ");   
}
```

Immutability 跟 value types 沒有絕對關係，struct 和 class 都可以是 Immutable 也可以是 Mutable。勿將兩者當做是同一件事。