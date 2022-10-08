# [C# 2.0]Nullable Type

讓實質型別(value type)也可以為 null
```csharp
Nullable<int> num1 = new Nullable<int>(10);
Nullable<int> num2 = null;
```
或是簡化的寫法
```csharp
int? num1 = 10;
int? num2 = null;
```

<br/>int? 無法直接指派給 int
```csharp
int num1 = 10;
int? num2 = 10;

// compiler error
num1 = num2;
```

<br/>硬給值也可能出 run time error
```csharp
int num1 = 10;
int? num2 = null;

// run time error
num1 = num2.Value;
```

<br/>最好先檢查
```csharp
int num1 = 10;
int? num2 = null;

if (num2.HasValue)
{
    num1 = num2.Value;
    num1 = (int)num2;
}
```
或是這樣檢查
```csharp
int num1 = 10;
int? num2 = null;

num1 = num2.GetValueOrDefault();
```

<br/>在執行指較時，也要考慮 null 的情況，不能認為
<br/>>= 不成立就代表 < 成立
<br/>!= 不成立就代表 == 成立
```csharp
int? num1 = 10;
int? num2 = null;

if(num1 >= num2)
{
    Console.WriteLine("num1 >= num2 returned true");
}
else
{
    Console.WriteLine("num1 >= num2 returned false, bug num < num2 is also false!!");
}
```
>num1 >= num2 returned false, bug num < num2 is also false!!

```csharp
int? num1 = 10;
int? num2 = null;

if(num1 < num2)
{
    Console.WriteLine("num1 < num2 returned true");
}
else
{
    Console.WriteLine("num1 < num2 returned false, bug num >= num2 is also false!!");
}
```
>num1 < num2 returned false, bug num >= num2 is also false!!

```csharp
int? num1 = 10;
int? num2 = null;

if(num1 != num2)
{
    Console.WriteLine("num1 != num2 returned true");
}
```
>num1 != num2 returned true

```csharp
int? num1 = null;
int? num2 = null;

if(num1 == num2)
{
    Console.WriteLine("num1 == num2 returned true when value of each is null");
}
```
>num1 == num2 returned true when value of each is null