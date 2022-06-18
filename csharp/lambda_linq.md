# [C# 3.0]lambda 到 LINQ

宣告一個類別
```csharp
public class Employee
{
    public string Name;
    public int Age;
}
```

<br/>和宣告集合
```csharp
var employeeList = new List<Employee>();
```

<br/>如果需求是取出員工名稱中有 "i" 的，且照年齡排序
<br/>用 linq 可這樣寫

```csharp
var names =
    from e in employeeList
    where e.Name.Contains("i")
    orderby e.Age
    select e.Name;
```

<br/>等同於以下寫法
```csharp
var names = employeeList.Where(e => e.Name.Contains("i"))
    .OrderBy(e => e.Age)
    .Select(e => e.Name);
```

<br/>如果想不開，開 C# 2.0 的匿名函式就這樣寫
```csharp
var anNames = employeeList.Where<Employee>(new Func<Employee, ool>(delegate(Employee employee)
    {
        return employee.Name.Contains("i");
    }))
    .OrderBy<Employee, int>(new Func<Employee, int>(delegate(Employee employee)
    {
        return employee.Age;
    }))
    .Select<Employee, string>(new Func<Employee, int, string>(delegate(Employee employee, int i)
    {
        return employee.Name;
    }));
```

<br/>如果想不開，開 C# 1.0 的 delegate 這樣寫
```csharp
var delegateNames = employeeList.Where<Employee>(new Func<Employee, bool>(MyWhereFunc))
    .OrderBy<Employee, int>(new Func<Employee, int>(MyOrderByFunc))
    .Select<Employee, string>(new Func<Employee, string>(MySelectFunc));

public bool MyWhereFunc(Employee employee)
{
    return employee.Name.Contains("i");
}

public int MyOrderByFunc(Employee employee)
{
    return employee.Age;
}

public string MySelectFunc(Employee employee)
{
    return employee.Name;
}
```

<br/>相比之下，lambda 和 linq 的寫法真的簡潔許多