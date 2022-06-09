# [C#]== Operator - Equality operator

== 運算子在 Value Type 和 Reference Type 的運作方式沒有不同
- 在 Value Type 等於是比較值，只要值一樣就回傳 true
- 在 Reference Type 是比較指到的記憶體位址，也就是說，就算兩個物件屬於同一個類別，且成員變數都完全相同，他們會因為記憶體位址不同而讓 == 運算子回傳 false

<br/>而 string 雖然屬於 Referenct Type，依 [MSDN 文件](http://msdn.microsoft.com/zh-tw/library/53k8ybth%28v=vs.90%29.aspx)有提到 == 運算子會在 string 會比較值而非比較記憶體位址

<br/>可以用以下例子出看出差別

```csharp
var str1 = "123";
var str2 = "123";
            
Console.WriteLine($"str1 == str2: {str1==str2}");
Console.WriteLine($"ReferenceEquals(str1, str2): {ReferenceEquals(str1, str2)}");
```
因為 C# 有 string pool，同樣的字串會指到 string pool 中同樣的字串，所以會印出：
>str1 == str2: True
><br/>ReferenceEquals(str1, str2): True

<br/>另一個例子，我們在中間增加字串的計算
```csharp
str1 = "123";
var temp1 = "1";
var temp2 = "23";
// 中間可能有其他程式碼
str2 = temp1 + temp2;

Console.WriteLine($"str1 == str2: {str1==str2");
Console.WriteLine($"ReferenceEquals(str1, str2): {ReferenceEquals(str1, str2)}");
```
當字串有計算時，Compiler 無法知道 str1, str2 是不是同樣的內容，因為不保證 temp1, temp2 在執行過程中，值會不會變，所以會分別指派記憶體去儲存值，所以執行起來兩者結果不一樣：
>str1 == str2: True
><br/>ReferenceEquals(str1, str2): False

<br/>再一個例子，使用 new 關鍵字
```csharp
str1 = "123";
str2 = new string("123");

Console.WriteLine($"str1 == str2: {str1==str2");
Console.WriteLine($"ReferenceEquals(str1, str2): {ReferenceEquals(str1, str2)}");
```
因為使用了 new 關鍵字，compiler 會指派另一塊記憶體給 str2，，所以執行起來兩者結果不一樣：
>str1 == str2: True
><br/>ReferenceEquals(str1, str2): False

---

<br/>在 stackoverflow 看到有趣的例子

```csharp
static void Main(string[] args)
{
    CheckEquality("a", "a");
    Console.WriteLine("----------");
    CheckEquality("a", "ba".Substring(1));
}

static void CheckEquality<T>(T value1, Tvalue2) where T : class
{
    Console.WriteLine("value1: {0}", value1);
    Console.WriteLine("value2: {0}", value2);
    Console.WriteLine("value1 == value2:      {0}", value1 == value2);
    Console.WriteLine("value1.Equals(value2): {0}", value1.Equals(value2));
    if (typeof(T).IsEquivalentTo(typeof(string)))
    {
        string string1 = (string)(object)value1;
        string string2 = (string)(object)value2;
        Console.WriteLine("string1 == string2:    {0}", string1 == string2);
    }
}
```
印出結果如下：
>value1: a
><br/>value2: a
><br/>value1 == value2:      True
><br/>value1.Equals(value2): True
><br/>string1 == string2:    True
><br/>----------
><br/>value1: a
><br/>value2: a
><br/>value1 == value2:      False
><br/>value1.Equals(value2): True
><br/>string1 == string2:    True

這是因為 CheckEquality 的參數是泛型，因為此參數視作 class，因為 == 運算子都是比較記憶體位址，除非明轉型成 string，才能讓 == 運算子套用到對 string 的重載