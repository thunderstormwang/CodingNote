# [C# 2.0]yield

## yeild return
yield return 的做法為當遇到符合條件的元素時，即刻將該元素回傳回上一層進行後續運算，後續運算完後再回到迴圈中找尋下一個元素。  
在MSDN上對於yield的使用上還有提到一些限制：
- 回傳類型必須為IEnumerable, IEnumerable<T>, IEnumerator, IEnumerator<T>
- 不可包含任何 ref 與 out 參數
- Lambda 運算式、匿名方法與 unsafe 區塊不可使用 yield

### 範例一

#### 回傳 List
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

```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 1, 2, 3, 4, 5, 

#### yield return
可以用 yield return 改寫

```csharp
public IEnumerable<int> Get()
{
    yield return 1;
    yield return 2;
    yield return 3;
    yield return 4;
    yield return 5;
}
```
```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 1, 2, 3, 4, 5, 


### 範例二

#### 回傳 List

```csharp
public IEnumerable<int> Get()
{
    var result = new List<int>();
    for (var i = 0; i < 10; i++)
    {
        if (i % 2 == 0)
        {
            result.Add(i);
        }
    }

    return result;
}
```
```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 0, 2, 4, 6, 8, 

#### Linq 作法
```csharp
public IEnumerable<int> Get()
{
    return Enumerable.Range(0, 9).Where(i => i % 2 == 0);
}
```
```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 0, 2, 4, 6, 8, 

#### yield return
```csharp
public IEnumerable<int> Get(IEnumerable<int> collection)
{
    for (var i = 0; i < 10; i++)
    {
        if (i % 2 == 0)
        {
            yield return i;
        }
    }
}
```
```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 0, 2, 4, 6, 8, 

---

## yeild break
使用 yield break 陳述式結束反覆項目。也就是跳離當前函式。  
使用 break 會跳離當前迴圈。  

### yield break
```csharp
    public static IEnumerable<int> Get()
    {
        for (var i = 0; i < 10; i++)
        {
            if (i > 5)
            {
                yield break;
            }

            yield return i;
        }

        for (var i = 11; i < 15; i++)
        {
            yield return i;
        }
    }
```
```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 0, 1, 2, 3, 4, 5,

### break
```csharp
    public static IEnumerable<int> Get()
    {
        for (var i = 0; i < 10; i++)
        {
            if (i > 5)
            {
                break;
            }

            yield return i;
        }

        for (var i = 11; i < 15; i++)
        {
            yield return i;
        }
    }
```
```csharp
foreach (var item in Get())
{
    Console.Write($"{item}, ");
}
```
> 0, 1, 2, 3, 4, 5, 11, 12, 13, 14, 