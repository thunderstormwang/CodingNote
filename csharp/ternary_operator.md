# [C#]?: Operator - Ternary(Conditional) Operator

C# includes a decision-making operator ?: which is called the conditional operator or ternary operator. It is the short form of the if else conditions.

>condition ? consequent : alternative

```csharp
int input = 5;
string classify;

classify = (input > 0) ? "positive" : "negative";
Console.WriteLinke($"classify = {classify}");
```

等同以下寫法

```csharp
int input = 5;
string classify;

if (input > 0)
{
    classify = "positive";
}
else
{
    classify = "negative";
}
Console.WriteLinke($"classify = {classify}");
```

皆是以下結果
>lassify = positive