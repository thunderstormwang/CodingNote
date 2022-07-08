# Design Pattern - Iterator

- [Design Pattern - Iterator](#design-pattern---iterator)
  - [概觀](#概觀)
  - [用一個類別去實作兩個介面](#用一個類別去實作兩個介面)
  - [用兩個類別去分別實作兩個介面](#用兩個類別去分別實作兩個介面)
  - [yield return](#yield-return)
  - [For 和 foreach 是不一樣的](#for-和-foreach-是不一樣的)

---
## 概觀
+ 提供一種方式能夠依序存取物件中的每一個元素，但不需要知道聚合物件(aggregate object)的內部細節。
+ 在C# 中可以用現有的功能來完成。
+ 實作或回傳IEnumerable 或IEnumerable<T>。
+ 在較高階的程式語言(C#、Java)皆以內建進語言中

在 C# 如果要自己實作，可繼承 IEnumrator 和 IEnumrable 介面

---
## 用一個類別去實作兩個介面
```csharp
public class MyIterator : IEnumerable<double>, IEnumerator<double>
{
    private List<double> _list = new List<double>() { 0, 1, 2, 3, 4, 5 };
    private int _index = -1;
    
    public double Current
    {
        get
        {
            return _list[_index];
        }
    }

    public IEnumerator<double> GetEnumerator()
    {
        // 傳回能逐一查看集合的物件
        Reset();
        return this;
    }

    public void Reset()
    {
        // 設定列舉值至它的初始位置, 在集合中第一個元素之前
        _index = -1;
    }

    public bool MoveNext()
    {
        _index++;
        return _index >= _list.Count ? false : true;
    }

    object IEnumerator.Current
    {
        // 因為 IEnumerator<T> 有繼承 IEnumerator, 你也得實作此方法
        get
        {
            return null;
        }
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        // 因為 IEnumerable<T> 有繼承 IEnumerator, 你也得實作此方法
        return null;
    }

    public void Dispose()
    {
        // 因為 IEnumerator<T> 有繼承 IDisposable, 你也得實作此方法
    }
}
```

<br/>client 端程式碼
```csharp
MyIterator myIterator = new MyIterator();
foreach (double item in myIterator)
{
    Console.Write(item + " ");
}
```

---
## 用兩個類別去分別實作兩個介面
此類別實作 IEnumerable
```csharp
public class MyEnumerator : IEnumerator<double>
{
    private List<double> _list = new List<double>();
    private int _index = -1;
    public MyEnumerator(List<double> list)
    {
        _list = list;
    }
    public double Current
    {
        get
        {
            return _list[_index];
        }
    }
    public bool MoveNext()
    {
        _index++;
        return _index >= _list.Count ? false : true;
    }
    public void Reset()
    {
        // 設定列舉值至它的初始位置, 在集合中第一個元素之前
        _index = -1;
    }
    object IEnumerator.Current
    {
        // 因為 IEnumerator<T> 有繼承 IEnumerator, 你也得實作此方法
        get
        {
            return null;
        }
    }
    public void Dispose()
    {
        // 因為 IEnumerator<T> 有繼承 IDisposable, 你也得實作此方法
    }
}
```

<br/>此類別實作 IEnumerable
```csharp
public class MyIterator : IEnumerable<double>
{
    private List<double> _list = new List<double>() { 0, 1, 2, 3, 4, 5 };
    public IEnumerator<double> GetEnumerator()
    {
        // 傳回能逐一查看集合的物件
        return new MyEnumerator(_list);
    }
    IEnumerator IEnumerable.GetEnumerator()
    {
        // 因為 IEnumerable<T> 有繼承 IEnumerator, 你也得實作此方法
        return null;
    }
}
```

<br/>client 端程式碼
```csharp
MyIterator myIterator = new MyIterator();
foreach (double item in myIterator)
{
    Console.Write(item + " ");
}
```

---
## yield return
C# 2.0 以後，可用 yield return，其它的 compiler 都幫你寫

```csharp
public class MyIterator : IEnumerable<double>
{
    private List<double> _list = new List<double>() { 0, 1, 2, 3, 4, 5 };
    public IEnumerator<double> GetEnumerator()
    {
        for (int i = 0; i < _list.Count; i++)
        {
            yield return _list[i];
        }
    }
    IEnumerator IEnumerable.GetEnumerator()
    {
        // 因為 IEnumerable<T> 有繼承 IEnumerator, 你也得實作此方法
        throw new NotImplementedException();
    }
}
```

<br/>client 端程式碼
```csharp
MyIterator myIterator = new MyIterator();
foreach (double item in myIterator)
{
    Console.Write(item + " ");
}
```
---
## For 和 foreach 是不一樣的
+ for 看 index
+ Foreach 則是回傳後，下次再進來，則從下一個地方開始，意即記住上次執行的地方(語法：yield return)