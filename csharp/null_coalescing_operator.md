# [C#]?? & ??= Operrator - Null-Coalescing Operator - Null 聯合運算子

The null-coalescing operator ?? returns the value of its left-hand operand if it isn't null; otherwise, it evaluates the right-hand operand and returns its result. The ?? operator doesn't evaluate its right-hand operand if the left-hand operand evaluates to non-null.

範例一:
```csharp
int? x = null;

// Set to y to the value of x if x is NOT null; otherwise, 
// if x = null, set y to -1
int y = x ?? -1;
Console.WriteLinke($"y = {y}");
```
>y = -1

<br/>範例二:
```csharp
public int? GetNullableInt()
{
    return null;
}

// Assign i to return value of the method if the methos's result is NOT null;
// otherwise, if the result is null, set i to the default value of int.
int i = GetNullableInt() ?? default(int);
Console.WriteLinke($"i = {i}");
```
>i = 0

<br/>範例三:
```csharp
public string GetStringValue()
{
    return null;
}

// Display the value of s if s is NOT null; otherwise,
// display the string "Unspecified"
string s = GetStringValue();
Console.WriteLinke(s ?? "Unspecified");
```
>Unspecified
---
#### C# 8.0 後可以使用 ??= 運算子

以下寫法
```csharp
variable ??= expression;
```
等同於
```csharp
if (variable is null)
{
    variable = expression;
}
```