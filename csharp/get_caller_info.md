# [C# 5.0]取得呼叫者資訊

```csharp
internal class Program
{
    public static void Main(string[] args)
    {
        MyFunc("Hi!");
        
        Console.WriteLine("Hello World!!");
    }

    public static void MyFunc(string msg, [CallerMemberName] string memberName = "", [CallerFilePath] string filePath = "",
        [CallerLineNumber] int lineNumber = 0)
    {
        Console.WriteLine("msg: {0}, memberName: {1}, filePath: {2}, lineNumber: {3}", msg, memberName, filePath, lineNumber);
    }
}
```
>msg: Hi!, memberName: Main, filePath: C:\Users\roody_wang\RiderProjects\ConsoleApp1\ConsoleApp1\Program.cs, lineNumber: 9