# [C# 4.0]Convariance and Contravariance

## 什麼是 Convariance 和 Contravariance?

先看個例子，底下是我們很習慣的用法，將較小的型別指派給較大的型別
```csharp
string str = "test";
object obj = str;
```

<br/>Covariance 指的是將較小的型別指派給較大的型別
```csharp
IEnumerable<string> strings = new List<string>();
IEnumerable<object> objects = strings;
```

<br/>Contravariance 指的是將較大的型別指派給較小的型別
```csharp
Action<object> actObject = SetObject;
Action<string> actString = actObject;

void SetObject(object o) { }
```

<br/>兼具兩者特性可稱作 variance，兩者特性都沒有可稱作 invariance。
<br/>在 C#，在以下情況支援 variance:
+ 在 array 支援 Covariance(C# 1.0)
+ 在委派(delegate) 支援 Covariance 和 Contravariance(C# 2.0)
+ 在泛型介面和泛型委派支援 Variance(C# 4.0)

<br/>

## array 支援 covariance
以下程式碼可被 build 且可執行，將字串陣列指派給 object 陣列
```csharp
object[] obj = new String[10];
```

<br/>以下程式碼可被 build 但執行時會出例外
```csharp
obj[0] = 5;
```
<br/>由此我們知道，array 的 covariance 不太安全，compiler 無法把關所有失敗的情境

<br/>

## delegate 支援 variance
宣告以下函式
```csharp
object GetObject() { return null; }
void SetObject(object obj) { }

string GetString() { return ""; }
void SetString(string str) { }
```

<br/>covariance，該 Func 的回傳值是 object，我們可以將回傳值為 string 的函式指派給它
```csharp
Func<object> del = GetString;
```

<br/>contravariance，Action 的參數型別是 string，我們可以將參數型別為 object 的函式指派給它
```csharp
Action<string> del2 = SetObject;
```

<br/>但是泛型委派直到 C# 4.0 才支擾
```csharp
Func<string> del3 = GetString;
Func<object> del4 = del3; // Compiler error here until C# 4.0.
```

<br/>

## 泛型介面和泛型委派支援 Variance
### 泛型介面 的 covariance
在 C# 4.0 後，這樣寫可以通過編繹
```csharp
IEnumerable<object> list = new List<string>();
```
<br/>而即使在 C# 4.0，這樣寫也是 build error
```csharp
List<object> list = new List<string>(); // build error
List<object> list = new List<int>(); // build error
```
原因是只有泛型介面和泛型委派才有支援 convariance，在 class 和 struct 並不支援。
<br>也只能用在 reference type，value type 不行。
<br>從 array 的 convariance 我們看到有危險的情境，那麼為什麼在 4.0 後也讓泛型介面也支援 convariance 呢?
<br/>可以看以下例子，宣告 Fruit 類別，而 Apple, Orange 都繼承自 Fruit
```csharp
public class Fruit {}

public class Apple : Fruit {}

public class Orange : Fruit {}
```
<br>將較小的型別指派給較大的型別是很自然的事
```csharp
Fruit fruit = new Apple();
Fruit fruit2 = new Orange();
```

<br>若泛型介面不支援 covariance，那以下程式碼會得到 compiler error
```csharp
List<Fruit> fruits = new List<Fruit>();
List<Apple> apples = new List<Apple>();
List<Orange> oranges = new List<Orange>();

// 在 C# 4.0 以前, 這幾行會編繹不過
fruits.AddRange(apples);
fruits.AddRange(oranges);
```
以中文來說，我們可以說一個蘋果是一個水果，一個橘子是一個水果。
<br/>套用到 List, 我們想將一袋蘋果加入到一袋水果或是一袋橘子加入到一袋水果，compiler 卻告訴我們不能這樣做，我們必須要一個一個加進去，像這樣
```csharp
foreach(var apple in apples)
{
    fruitList.Add(apple);
}
```
這做法挺不直覺，所以 4.0 後就讓泛型介面也支援 convariance，但要如何避免陣列會發生的情境呢?
<br/>可以看 List.AddRange 的原型宣告
<br/>在 C# 4.0 以前是
```csharp
public void AddRange(IEnumerable<T> collection)
```
在 C# 4.0 以後是
```csharp
public void AddRange(IEnumerable<out T> collection)
```
多了 out 關鍵字，compiler 會檢查該 interface 提供的函式是否只將 T 作為回傳型別，若有將 T 作為參數型別，即過不了 compiler。
<br/>而 IEnumerable<T> 只提供了 GetEnumerator，這代表程式操作 IEnumerable<T> 時，只能取出，無法針對該集合做改變(增加/修改/刪除其中元素)，也就保證了安全
```csharp
public interface IEnumerable<out T> : IEnumerable
{
    IEnumerable<T> GetEnumerator();
}
```

<br/>

### 泛型介面 的 contravariance
有了 covariance 的例子，就不難理解 contravariance。
<br/>一樣沿用水果的例子，這次加了甜度的屬性
```csharp
public class Fruit 
{
    public int SweetDegree { get; set; }
}

public class Apple : Fruit {}

public class Orange : Fruit {}
```

```csharp
List<Apple> apples = new List<Apple>()
{
    new Apple() { SweetDegree = 45 },
    new Apple() { SweetDegree = 60 },
    new Apple() { SweetDegree = 75 },
}

IComparer<Fruit> cmp = new FruitComparer();
apples.Sort(cmp);
```
```csharp
public class FruitComparer : IComparer<Fruit>
{
    public int Compare(Fruit x, Fruit y)
    {
        return x.SweetDegree - y.SweetDegree;
    }
}
```
Sort 接受的是 IComparer<Apple>，但也接受我們將 IComparer<Fruit> 指派給它。
<br/>在 C# 4.0 以前，得為 Apple, Orange 各寫一個 Comparer，即使該屬性是繼承自 Fruit。所以 4.0 後就讓泛型介面也支援 contravariance。
<br/>可以看 List.Sort 的原型宣告
<br/>在 C# 4.0 以前是
```csharp
public void Sort(IComparer<T> comparer)
```
在 C# 4.0 以後是
```csharp
public void Sort(IComparer<in T> comparer)
```
多了 in 關鍵字，compiler 會檢查該 interface 提供的函式是否只將 T 作為參數型別，若有將 T 作為回傳型別，即過不了 compiler。

<br/>

### 預設支援 variance 的泛型介面
+ [IEnumerable\<T> (T is covariant)
+ IEnumerator\<T> (T is covariant)
+ IQueryable\<T> (T is covariant)
+ IGrouping\<TKey, TElement> (TKey and TElement are covariant)
+ IComparer\<T> (T is contravariant)
+ IEqualityComparer\<T> (T is contravariant)
+ IComparable\<T> (T is contravariant)

<br/>

### 預設支援 variance 的泛型委派
+ Action delegates from the System namespace, for example, Action\<T> and Action\<T1, T2> (T, T1, T2, and so on are contravariant)
+ Func delegates from the System namespace, for example, Func\<TResult> and + Func\<T, TResult> (TResult is covariant; T, T1, T2, and so on are contravariant)
+ Predicate\<T> (T is contravariant)
+ Comparison\<T> (T is contravariant)
+ Converter\<TInput, TOutput> (TInput is contravariant; TOutput is covariant.)

<br/>

## 實作自己的 convariance 和 contravariace 的 interface
+ 關鍵字 out 代表 convariance，將較小的型別 assign 給較大的型別，只能用在回傳值上
+ 關鍵字 in 代表 contravariance，將較大的型別 assign 給較小的型別，只能用在輸入的參數上

```csharp
interface IVariant<out R, in A>
{
    // These methods satisfy the rules.
    R GetR();
    void SetA(A sampleArg);
    R GetRSetA(A sampleArg);

    // And these don’t.
    // A 不能當傳型別
    // R 不能當參數型別
    A GetA();
    void SetR(R sampleArg);
    A GetASetR(R sampleArg);
}
```

<br/>out, in 沒有繼承關係，你需再加上關鍵字才能繼續擁有此特性
```csharp
public interface IParent<out>
{}

public interface IChild<out T> : IParent<T>
{}
```

<br/>如前所述，class 不支援
```csharp
// compiler error
public class SomeClass<out T> :  IParent<out>
{}
```
> Invlid variance modifier, Only interface and delegate type of parameters can be specified as variant.

---
參考自
+ [C# 4.0：Covariance 與 Contravariance 觀念入門](https://www.huanlintalk.com/2009/10/c-40covariance-and-contravariance.html) 
+ [Convariance and Contravariance FAQ](https://docs.microsoft.com/en-us/archive/blogs/csharpfaq/covariance-and-contravariance-faq)