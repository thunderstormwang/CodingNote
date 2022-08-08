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
