# [C# 3.0]SelectMany

SelectMany 為 C# 3.0 的一個擴充方法，作用很像攤平。

創造兩個類別，Teacher 和 Student。
```csharp
public class Student
{
    public int Score { get; set; }

    public Student(int score)
    {
        Score = score;
    }
}
```
```csharp
public class Teacher
{
    public string Name { get; set; }

    public List<Student> Students { get; set; }

    public Teacher(string name, List<Student> students)
    {
        Name = name;
        Students = students;
    }
}
```
再建以下集合，有7個老師，每個老師底下各有3個學生，共有3個學生不及格
```csharp
List<Teacher> teachers = new List<Teacher>
{
    new Teacher("a", new List<Student> { new Student(100), new Student(90), new Student(30) }),
    new Teacher("b", new List<Student> { new Student(100), new Student(90), new Student(60) }),
    new Teacher("c", new List<Student> { new Student(100), new Student(90), new Student(40) }),
    new Teacher("d", new List<Student> { new Student(100), new Student(90), new Student(60) }),
    new Teacher("e", new List<Student> { new Student(100), new Student(90), new Student(50) }),
    new Teacher("f", new List<Student> { new Student(100), new Student(90), new Student(60) }),
    new Teacher("g", new List<Student> { new Student(100), new Student(90), new Student(60) })
};
```

<br/>如果我們要取出不及格的學生，在 C# 2.0 可以用二重 foreach
```csharp
List<Student> studentList = new List<Student>();
foreach (var t in teachers)
{
    foreach (var s in t.Students)
    {
        if (s.Score < 60)
        {
            studentList.Add(s);
        }
    }
}
```
C# 3.0 可以這樣用，一行解決
```csharp
var studentList = teachers.SelectMany(t => t.Students).Where(s => s.Score < 60);
```

---

## 跟 Select 的差別
取出來的型別不一樣，數量也不同
- Select 出來跟原本數量會一樣
- SelectMany 出來會攤平，跟原本數量不一樣

```csharp
IEnumerable<List<Student>> selectlist = teachers.Select(t => t.Students);
IEnumerable<Student> selectManylist = teachers.SelectMany(t => t.Students);

Console.WriteLine($"selectlist.Count = {selectlist.Count()}");
Console.WriteLine($"selectManylist.Count = {selectManylist.Count()}");
```
>selectlist.Count = 7
<br/>selectManylist.Count = 21

---

## 進階用法
得到一個匿名物件集合，內有不及格學生的老師名字、學生分數

```csharp
var list = teachers
            .SelectMany(t => t.Students, (a, b) => new { a.Name, b.Score})
            .Where(s => s.Score < 60);
```