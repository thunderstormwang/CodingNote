# [C#]enum type

- [[C#]enum type](#cenum-type)
  - [string, enum conversion](#string-enum-conversion)
  - [列舉, 數字轉換](#列舉-數字轉換)
  - [列出所有名稱](#列出所有名稱)
  - [列出所有值](#列出所有值)
  - [搭配 attribute 實作回傳代碼](#搭配-attribute-實作回傳代碼)
  - [Enumeration class vs Enumeration Typs](#enumeration-class-vs-enumeration-typs)

<br/>

## string, enum conversion
定義這樣的列舉
```csharp
public enum Color
{
    Red = 0,
    Green = 1,
    Blue = 2
}
```

<br/>將列舉轉為字串
```csharp
var color = Color.Red;
Console.WriteLine($"Enum To String: {day.ToString()}");
```
> Enum To String: Red

<br/>將字串轉為列舉
```csharp
color = (Color)Enum.Parse(typeof(Color), "green", true);
Console.WriteLine($"String To Enum: {day}");
```
> String To Enum: Green

<br/>字串轉換若發生對不上的情況會丟出例外
```csharp
try
{
    var color = Enum.Parse(typeof(Color), "Orange");
}
catch (Exception ex)
{
    Console.WriteLine($"Error when Enum.Parse: {ex.Message}" );
}
```
> Error when Enum.Parse: 找不到要求的值 'Orange'。
---

## 列舉, 數字轉換
將列舉轉為數字
```csharp
Console.WriteLine($"Enum to Int: {(int)Color.Green}");
```
> Enum to Int: 1

<br/>將數字轉回列舉
```csharp
var color = (Color)2;
Console.WriteLine($"Int to Enum: {color}");
```
> Int to Enum: Blue

<br/>數字轉換若發生對不上的情況不會丟出例外，但會出現非列舉值
```csharp
color = (Color)100;
Console.WriteLine($"Int(100) to Enum: {color}");
```
> Int(100) to Enum: 100
---

## 列出所有名稱
列出所有名稱
```csharp
foreach (var item in  Enum.GetNames(typeof(Color)))
{
    Console.WriteLine($"Enum Name: {item}");
}
```
> Enum Name: Red
<br/>Enum Name: Green
<br/>Enum Name: Blue
---

## 列出所有值
用 extension method
```csharp
public static class EnumUtil
{
    public static IEnumerable<T> GetValues<T>()
    {
        return Enum.GetValues(typeof(T)).Cast<T>();
    }
}
```

<br/>列出所有值
```csharp
foreach (var item in EnumUtil.GetValues<Color>())
{
    Console.WriteLine($"Enum Value: {(int)item}");
}
```
> Enum Value: 0
<br/>Enum Value: 1
<br/>Enum Value: 2
---

## 搭配 attribute 實作回傳代碼
定義回傳代碼如下
```csharp
public enum ResultCodeEnums
{
    [Description("Success")]
    Success = 0,
    
    [Description("Action Error")]
    ActionError = 1002,

    [Description("Parameter Error")]
    ParameterError = 1003
}
```

<br/>用 extension method 取得各個回傳代碼的說明
```csharp
public static class EnumHelper
{
    public static string GetEnumDescription(this Enum value)
    {
        var customAttributes = (DescriptionAttribute[])value.GetType().GetField(value.ToString())
            ?.GetCustomAttributes(typeof(DescriptionAttribute),
                false);
        return customAttributes != null && customAttributes.Length > 0
            ? customAttributes.First().Description
            : value.ToString();
    }
}
```

<br/>如此回傳代碼跟訊息就可定義在同個地方了
```
var resultCode = ResultCodeEnums.ParameterError;
var errorMessage = resultCode.GetEnumDescription();
```

---

## Enumeration class vs Enumeration Typs

[MSDN](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/enumeration-classes-over-enum-types) 推薦使用 Enumeration class, 理由是可以使用物件導向的功能, 但沒給實際例子.
<br/>看了
[此篇](https://lostechies.com/jimmybogard/2008/08/12/enumeration-classes/), 我才比較有概念

<br/>以下這種 model 很常見到
```csharp
public class Employee
{
    public EmployeeType Type { get; set; }
}

public enum EmployeeType
{
    Manager,
    Servant,
    AssistantToTheRegionalManager
}
```

<br/>這樣的寫法會伴隨著許多 switch case, 例如
```csharp
public void ProcessBonus(Employee employee)
{
    switch(employee.Type)
    {
        case EmployeeType.Manager:
            employee.Bonus = 1000m;
            break;
        case EmployeeType.Servant:
            employee.Bonus = 0.01m;
            break;
        case EmployeeType.AssistantToTheRegionalManager:
            employee.Bonus = 1.0m;
            break;
        default:
            throw new ArgumentOutOfRangeException();
    }
}
```

<br/>這會造成幾個問題
- Behavior related to the enumeration gets scattered around the application
- New enumeration values require shotgun surgery
- Enumerations don’t follow the Open-Closed Principle

我的理解是
- 因為 enum type 裡不能有函式, 只能將邏輯寫在其它地方, 就算是寫成 extension method, 也必須寫在其它地方, 使用商業邏輯四散各地.
- 當你新增 1 個新的 enum value, 例如 HR 後, 因為第一項的關係, 你必須尋找很多地方去補上商業邏輯(補上 swtich case), 以上例來說, 你不走增加 case，會讓程式用 HR 走到 default 然後就丟例外了.
- 也因為第二項的關係, 你每增加一個 value, 就得對原有的程式做修改，這就不符合對擴展開放，對修改封閉的原則了

<br/>如果我們改用 enumeration calss
```csharp
public abstract class Enumeration : IComparable
{
    public int Id { get; private set; }
    public string Name { get; private set; }

    protected Enumeration(int id, string name) => (Id, Name) = (id, name);

    public override string ToString() => Name;

    public static IEnumerable<T> GetAll<T>() where T : Enumeration =>
        typeof(T).GetFields(BindingFlags.Public |
                            BindingFlags.Static |
                            BindingFlags.DeclaredOnly)
                    .Select(f => f.GetValue(null))
                    .Cast<T>();

    public override bool Equals(object obj)
    {
        if (obj is not Enumeration otherValue)
        {
            return false;
        }

        var typeMatches = GetType().Equals(obj.GetType());
        var valueMatches = Id.Equals(otherValue.Id);

        return typeMatches && valueMatches;
    }

    public override int GetHashCode() => Id.GetHashCode();

    public static int AbsoluteDifference(Enumeration firstValue, Enumeration secondValue)
    {
        var absoluteDifference = Math.Abs(firstValue.Id - secondValue.Id);
        return absoluteDifference;
    }

    public static T FromValue<T>(int value) where T : Enumeration
    {
        var matchingItem = Parse<T, int>(value, "value", item => item.Id == value);
        return matchingItem;
    }

    public static T FromDisplayName<T>(string displayName) where T : Enumeration
    {
        var matchingItem = Parse<T, string>(displayName, "display name", item => item.Name == displayName);
        return matchingItem;
    }

    private static T Parse<T, K>(K value, string description, Func<T, bool> predicate) where T : Enumeration
    {
        var matchingItem = GetAll<T>().FirstOrDefault(predicate);

        if (matchingItem == null)
            throw new InvalidOperationException($"'{value}' is not a valid {description} in {typeof(T)}");

        return matchingItem;
    }

    public int CompareTo(object other) => Id.CompareTo(((Enumeration)other).Id);

    // Other utility methods ...
}
```
<br/>再繼承 Enumeration 去實作

```csharp
public class EmployeeType : Enumeration
{
    public static EmployeeType Manager = new EmployeeType(1, "Manager");
    public static EmployeeType Servant = new EmployeeType(2, "Servant");
    public static EmployeeType AssistantToTheRegionalManager = new EmployeeType(3, "Assistant To The Regional Manager");

    public EmployeeType(int id, string name)
        : base(id, name)
    { }

    public static IEnumerable<EmployeeType> List() =>
        new[] { Manager, Servant, AssistantToTheRegionalManager };

    public static EmployeeType From(int id)
    {
        var state = List().SingleOrDefault(s => s.Id == id);
        return state ?? null;
    }
}
```

<br/>雖然變成 class, 原本的 model 的用法不變
```csharp
public class Employee
{
    public EmployeeType Type { get; set; }
}

var employee = new Employee();
employee.Type = EmployeeType.Manager;
```

<br/>而且因為 EmployeeType 是類別, 我們可以在裡面增加商業邏輯, 例如實作 BonusSize, 依據不同的 EmployeeType 回傳不同的 BonusSize
```csharp
public void ProcessBonus(Employee employee)
{
    employee.Bonus = employee.Type.BonusSize;
}
```
如此讓商業邏輯不會四散在各地, 連 switch case 也不需要了(但是得多寫 code....)

<br/>甚至可進一步將 EmployeeType 變成父類別，將 BonusSize 的實作給分開
```csharp
public abstract class EmployeeType : Enumeration
{
    public static readonly EmployeeType Manager = new ManagerType();

    protected EmployeeType() { }
    protected EmployeeType(int value, string displayName) : base(value, displayName) { }

    public abstract decimal BonusSize { get; }

    private class ManagerType : EmployeeType
    {
        public ManagerType() : base(0, "Manager") { }

        public override decimal BonusSize
        {
            get { return 1000m; }
        }
    }
}
```