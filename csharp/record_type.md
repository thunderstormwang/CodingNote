# [C# 9.0]record type

C# 9.0 增加了新型態 record(可參考 [MSDN](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/builtin-types/record)), 簡化了 reference type 在達成 immutable 的程式碼

<br/>想像以下情境, 我們有個類別 Member

```csharp
public class Member
{
    public int Id { get; init; }
    public string Name { get; init; }
}
```

<br/>並且用它創造新物件

```csharp
var member = new Member()
{
    Id = 1,
    Name = "John"
};
Console.WriteLine($"Id: {member.Id}, Name: {membName}");
```

<br/>如果我們要創造另一個物件, 且只有 Name 不一樣時, 我們會這樣做

```csharp
var newMember = new Member()
{
    Id = member.Id,
    Name = "Mary"
};
Console.WriteLine($"Id: {member.Id}, Name: {membName}");
```

<br/>想像一下如果 Member 裡面有數十個 property 的話, 這將是個繁瑣的複製貼上...
<br/>在 C# 9.0 後, 我們可以用 recrod type 搭配 with 關鍵字簡化此過程

```csharp
public record Member
{
    public int Id { get; init; }
    public string Name { get; init; }
}
```

```csharp
var member = new Member()
{
    Id = 1,
    Name = "John"
};
Console.WriteLine($"Id: {member.Id}, Name: {membName}");

var newMember = member with { Name = "Mary" };
Console.WriteLine($"Id: {member.Id}, Name: {membName}");
```

<br/>另一個 record 跟 class 不一樣的地方是
- 在 class 的物件, 如果屬於同個型別且記憶體位置一樣, 會被判斷相等
- 在 record 的物件, 如果屬於同個型別且儲存相同的值, 會被判斷相等

因為 compiler 會對 record 做以下動作
- 的覆寫 Object.Equals(Object)
- 當兩個參數都不是 null 時，會使用這個方法作為靜態方法的基礎 Object.Equals(Object, Object)
- 其參數為記錄類型的虛擬 Equals 方法。 這個方法會實作 IEquatable<T>
- 覆寫 Object.GetHashCode()
- 運算子 == 和 != 的覆寫。

<br/>例如以下範例:

```csharp
var member = new Member()
{
    Id = 1,
    Name = "John"
};

var newMember = new Member()
{
    Id = member.Id,
    Name = member.Name
};

Console.WriteLine($"member == newMember: {member newMember}");
Console.WriteLine($"ReferenceEquals(member, newMemb: {ReferenceEquals(member, newMember)}");
```

<br/>會印出以下結果
<br/>即使這兩個物件的記憶體位置不一樣, 在 == 仍會因為值都相同而被判定相同
>member == newMember: True
<br/>ReferenceEquals(member, newMember): False


```csharp
var member = new Member()
{
    Id = 1,
    Name = "John"
};

var newMember = new Member()
{
    Id = member.Id,
    Name = "Mary"
};

Console.WriteLine($"member == newMember: {member newMember}");
Console.WriteLine($"ReferenceEquals(member, newMemb: {ReferenceEquals(member, newMember)}");
```

<br/>所以如果值不一樣, 就會印出以下結果
>member == newMember: False
<br/>ReferenceEquals(member, newMember): False