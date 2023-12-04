# [C# 4.0]dynanmic type - 動態型別

告訴 compiler 在 compile time 不要檢查型別，因為在 compile time 時你也不知道其型別，但此物件在 run time 時就知道型別(所以程式在執行階段可能因型別不符得到例外)

```csharp
dynamic variable = "10";
Console.WriteLine(variable.GetType());
```
>System.String

<br/>中途指派另一個型別也是可以的
```csharp
variable = 10;
Console.WriteLine(variable.GetType());
```
>System.Int32

<br/>後續也可接著做運算
```csharp
variable += 10;
Console.WriteLine(variable);
```
>20

<br/>會自動換型別
```csharp
dynamic variable = 10;
Console.WriteLine(variable.GetType());
variable += "100";
Console.WriteLine(variable.GetType());
```
>System.Int32
<br/>System.String

<br/>像 JavaScript 一樣可以為物件動態加入新成員(屬性或函式)，ASP.NET MVC 的 ViewBag 也是 dynamic，讓你可將物件傳給 View
```csharp
dynamic sampleObject = new ExpandoObject();

sampleObject.Title = "Hello World!!";
sampleObject.Price = 1350;

Console.WriteLine($"Title: {sampleObject.Title}, GetType: {sampleObject.Title.GetType()}");
Console.WriteLine($"Title: {sampleObject.Price}, GetType: {sampleObject.Price.GetType()}");
```
>Title: Hello World!!, GetType: System.String
<br/>Title: 1350, GetType: System.Int32

<br/>dynamic 還可應用在
- 可簡化對 COM API (例如 Office 自動化 API)、動態 API (例如 IronPython 程式庫) 及 HTML 文件物件模型 (DOM) 的存取。
- Silverilght與JavaScript間的互動

---
## The difference between var and dynamic keywords

var 在 compile time 就會決定其型別，之後也無法再指派為另一個型別，但 dynamic 可以
```csharp
var sampleObject = 10;

// compiler error
sampleObject = "Hello";
```
>[CS0029] 無法將類型 'string' 隱含轉換成 'int'

<br/>dynamic 允許這樣的轉換
```csharp
dynamic sampleObject = 10;
sampleObject = "Hello";
```

<br/>C# 3.0 的匿名物件，在宣告後就不能增加成員，dynamic 可以
```csharp
var sampleObject = new 
{
    Title = "Hello World!!"
};

// compiler error
sampleObject.Price = 1350;
```
>[CS1061] '<anonymous type: string Title>' 未包含 'Price' 的定義，也找不到可接受類型 '<anonymous type: string Title>' 第一個引數的可存取擴充方法 'Price' (是否遺漏 using 指示詞或組件參考?)

<br/>dynamic 允許這樣的作法
```csharp
dynamic sampleObject = new ExpandoObject();

sampleObject.Title = "Hello World!!";
sampleObject.Price = 1350;
```

<br/>var 有 intellisense，但 dynamic 沒有

---
## The difference between dynamic and object keywords

抄自 [MSDN網誌](https://docs.microsoft.com/en-us/archive/blogs/csharpfaq/what-is-the-difference-between-dynamic-and-object-keywords)


<br/>很理所當然地印出 int32，因為這是儲存在 object 的型態
```csharp
object obj = 10;
Console.WriteLine(obj.GetType());
```
>System.Int32

<br/>如果我們直接做加法運算如以下程式碼，會得到 compiler error，因為 object 並不支援這樣的運算
```csharp
// compiler error
obj = obj + 10;
```
需要做明確的轉型
```csharp
obj = (int)obj + 10;
```
但即使做了轉型也不代表就安全，因為你可能指定錯誤的型別
```csharp
// runtime error
obj = (string)obj + 10;
```
>Unhandled exception. System.InvalidCastException: Unable to cast object of type 'System.Int32' to type 'System.String'.

<br/>即使你轉成其它數值型別，而且該語言支援隱含轉型，仍會出例外
```csharp
// runtime error
obj = (double)obj + 10;
```
>Unhandled exception. System.InvalidCastException: Unable to cast object of type 'System.Int32' to type 'System.Double'.

<br/>換來看看 dynamic 的情況
<br/>一樣很理所當然地印出 int32，跟 object 情況相同
```csharp
dynamic dyn = 10;
Console.WriteLine(dyn.GetType());
```
>System.Int32 

如果我們直接做加法運算如以下程式碼，是不會出錯，因為在 compiler 不會去辨認該型別
```csharp
// No compiler error
dyn = dyn + 10;
```
甚至加法運算可以用在所有有支援的數值型別上
```csharp
// No compiler error
dyn = 10.0;
dyn = dyn + 10;

dyn = "10";
dyn = dyn + 10;
```

### 用在函式的參數
```csharp
public static void Print(string arg)
{
    Console.WriteLine(arg);
}
```

```csharp
public static void Print(string arg)
{
    Console.WriteLine(arg);
}
```

```csharp
dynamic dyn = 10;

// You get an exception at run time here.
Print(dyn);

dynamic dyn = "10";
Print(dyn);
```

```csharp
object obj = 10;

// compiler error.
//Print(obj);

// Compiles, but there is an exception at run time.
//Print((string)obj);

// This code works because obj is now a string,
// but you still need a cast.
obj = "10";
Print((string)obj);
```

<br/>結論：使用 dynamic 並沒有更安全或更危除，只要你型別錯誤，仍會在 run time 得到錯誤，它的好處為簡化某些程式的寫法