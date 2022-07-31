# [C# 9.0]Init-Only property

以往在 C# 想創造不可變(immutable)的 property, 可以這樣寫
<br/>在初始化給過值後, 就不能再改變值

```csharp
public class Member
{
    public int Id { get; set; }
    public string Name { get; }
    
    public Member(string name)
    {
        Name = name;
    }
}
```

```csharp
var member = new Member("John");
member.Id = 1;

Console.WriteLine($"Id: {member.Id}, Name: {membName}");
```

<br/>在 C# 9.0 引入了 Init-Only property, 多一種寫法

```csharp
public class Member
{
    public int Id { get; set; }
    public string Name { get; init; }
}
```

```csharp
var member = new Member
{
    Name = "John"
};

member.Id = 
Console.WriteLine($"Id: {member.Id}, Name: {membName}");
```

<br/>想在建構子賦值給 Init-Only property 也是可以的

```csharp
public class Member
{
    public int Id { get; set; }
    public string Name { get; init; }
    
    public Member(string name)
    {
        Name = name;
    }
}
```

```csharp
var member = new Member("John");
member.Id = 1;

Console.WriteLine($"Id: {member.Id}, Name: {membName}");
```