# [C# 3.0]Object and Collection Initializer

## Object Initializer

一個簡單的類別
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

<br/>C# 3.0 以前, 要一個一個 property 指定初始值
```csharp
var person1 = new Person();
person1.Name = "John";
person1.Age = 28;
```

<br/>C# 3.0 以後可以用這種方式初始化, compiler 會幫你轉上面的語法
```csharp
var person1 = new Person()
{
    Name = "John",
    Age = 28
};
```

---

## Collection Initializer

```csharp
var numbers = new List<int>()
{
    1, 2, 3, 4, 5
}
```

```csharp
var people = new List<Person>()
{
    new Person { Name = "John", Age = 28 },
    new Person { Name = "Mary", Age = 25 },
}
```

```csharp
var employDict = new Dictionary<string, string>()
{
    { "001", "John" },
    { "002", "Mary" },
}
```