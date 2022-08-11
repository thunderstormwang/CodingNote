# [C# 3.0]Anonymous Type - 匿名型別

使你省略類別定義，由 compiler 幫你產生型別名稱。
- 成員可為一個或多個 public 唯讀的 property，不能有方法或事件
- 支援 intellisense
- 物件產生出來後，無法再被指派新值

<br/>compiler 會幫你產生型別名稱，但無法使用此名字去宣告變數
```csharp
var anonymousTyp1 = new { Name = "John", Age = 30 };

Console.WriteLine(anonymousTyp1.GetType());
```
><>f__AnonymousType0`2[System.String,System.Int32]


<br/>同樣的成員會使用相同的型別名稱
```csharp
var anonymousTyp2 = new { Name = "Mary", Age = 28 };

Console.WriteLine(anonymousTyp2.GetType());
```
><>f__AnonymousType0`2[System.String,System.Int32]

<br/>不同的成員會使用其它的型別名稱，代表是不同型別
```csharp
var anonymousTyp3 = new { N = "John", A = 30 };

Console.WriteLine(anonymousTyp3.GetType());
```
><>f__AnonymousType1`2[System.String,System.Int32]

<br/>物件產生出來後，無法再被指派新值
```csharp
var anonymousTyp1 = new { Name = "John", Age = 30 };

// compiler error
anonymousTyp1.Name = "Peter"
```
>[CS0200] 無法指派為屬性或索引子 '<anonymous type: string Name, int Age>.Name' -- 其為唯讀

---

## 注意事項
- compiler 沒有為匿名型別實作 IDisposable 介面，因此你應該避免在匿名型別中存放可丟棄的(disposable)物件。
- 不能當作參數傳遞，也不應該放進如 List 之類的 collection。
- 承上，你可轉型成 object 做傳遞，這可以被 compile，但沒有意義，因為轉型不回來，也失去 intellisense