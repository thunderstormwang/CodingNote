# [C# 2.0]yield

## yeild return
yield return 的做法為當遇到符合條件的元素時，即刻將該元素回傳回上一層進行後續運算，後續運算完後再回到迴圈中找尋下一個元素。
在MSDN上對於yield的使用上還有提到一些限制：

- 回傳類型必須為IEnumerable, IEnumerable<T>, IEnumerator, IEnumerator<T>
- 不可包含任何 ref 與 out 參數
- Lambda 運算式、匿名方法與 unsafe 區塊不可使用 yield

### 範例一
<br/>yield return
```csharp
public IEnumerable<int> Get()
{
    yeild return 1;
    yeild return 2;
    yeild return 3;
    yeild return 4;
    yeild return 5;
}
```

<br/>回傳 List
```csharp
public IEnumerable<int> Get()
{
    IEnumerable<int> result = new List<int>()
    {
        1, 2, 3, 4, 5
    };

    return result;
}
```

### 範例二
<br/>yield return
```csharp
public IEnumerable<int> Get(IEnumerable<int> collection)
{
    return collection.ToList().Where(i => i % 2 == 0);
}
```

<br/>Linq 作法
```csharp
public IEnumerable<int> Get()
{
    IEnumerable<int> result = new List<int>()
    {
        1, 2, 3, 4, 5
    };

    return result;
}
```

<br/>回傳 List
```csharp
public IEnumerable<int> Get()
{
    IEnumerable<int> result = new List<int>();
    foreach(var i in collection)
    {
        if(i % 2 == 0) 
        {
            result.Add(i);
        }
    }
    return result;
}
```


---

## yeild break
使用 yield break 陳述式結束反覆項目。
