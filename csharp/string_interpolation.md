# [C# 6.0]String interpolation

6.0組字串的新語法，看起來更直覺

```csharp
var foo = 34;
var bar = 42;

var resultString = $"The foo is {foo}, and the bar is {bar}.";
Console.WriteLinke(resultString);
```
>The foo is 34, and the bar is 42.";

<br/>其實是 compiler 幫你轉成下列語法
```csharp
var resultString = string.Format("The foo is {0}, and the bar is {1}.", new object [] { foo, bar });
```

<br/>如果需要印出大括孤, 就重覆一次大括孤
```csharp
var resultString = $"The {{foo}} is {foo}, and the {{bar}} is {bar}.";
Console.WriteLinke(resultString);
```
>The {foo} is 34, and the {bar} is 42.";


<br/>string interpolation 和 verbatim string literals 可以合併使用
```csharp
var foo = 34;
var bar = 42;

var resultString = $@"In case it wasn't clear:
\u00B9
The foo
is {foo},
and the bar
is {bar}.";
Console.WriteLinke(resultString);
```
>"In case it wasn't clear:
<br/>\u00B9
<br/>The foo
<br/>is {foo},
<br/>and the bar
<br/>is {bar}.

<br/>也可以在大括號內指定輸出格式
```csharp
var bar = 42;

Console.WriteLinke($"{{bar}} is {bar:D6}");
```
>{bar} is 000042.";

<br/>指定日期格式
```csharp
var bar = 42;

Console.WriteLinke($"DateTime.Now is {DateTime.Now:yyyy/MM/dd}");
```
>DateTime.Now is 2017/08/29";

<br/>也支援邏輯運算
```csharp
var foo = 34;
var bar = 42;

Console.WriteLinke($"the bigger one is {Math.Max(foo, bar)}");
Console.WriteLinke($"the bigger one is {foo >= bar ? foo : bar}");
Console.WriteLinke($"2 + 2 = {2 + 2}");
```
>the bigger one is 42
<br/>the bigger one is 42
<br/>2 + 2 = 4