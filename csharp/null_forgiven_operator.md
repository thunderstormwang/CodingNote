# [C# 8.0]! Operrator - Null-Forgiven Operator

這只能壓制 compiler warning，在 run time 時沒有效果
<br/>儘量少用，這可能造成難以發現的 bug
<br/>可以用在:
+ In some (edge) cases, the compiler is not able to detect that a nullable value is actually non-nullable.
+ Easier legacy code-base migration.
+ In some cases, you just don't care if something becomes null.
+ When working with Unit-tests you may want to check the behavior of code when a null comes through.

```csharp
string x;
string? y;

x = y!;
```

```csharp
var teachers = new List<Teacher>;

string name = teachers.FirstOrDefault()!.Name;
```