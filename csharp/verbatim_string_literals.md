# [C#]verbatim string literals，字串前綴 + @

以往的字串指定方式, 得用「\」控制特殊字元，例如換行、tab、雙引號
```csharp
var str = "line1\r\nHelloWorld! \t \"John\"";
Console.WriteLine(str);
```
>line1
<br/>HelloWorld!      "John"

<br/>加上前綴 @ 後, 會保留格式，例如換行、tab、只有雙引號需要特別處理
```csharp
var str = @"line1
HelloWorld!      ""John""";
Console.WriteLine(str);
```
>line1
<br/>HelloWorld!      "John"