# [C# 3.0]Extension Method 擴充方法

Extension methods enable you to "add" methods to existing types without creating a new derived type, recompiling, or otherwise modifying the original type. Extension methods are static methods, but they're called as if they were instance methods on the extended type.

C# 3.0 後，提供擴充方法
<br/>提供可以擴充 Legacy Code，又不會動到舊的 Code 的選擇

<br/>首先宣告靜態類別, 函式也須是靜態

```csharp
namespace MyExtensions
{
    public static class UtilityExtensions
    {
        // 第一個參數須加上 this 修飾詞,
        // 讓 compiler 知道 this 後面接的型別是該方法欲擴充的類別
        public static string Hello(this string name)
        {
            return $"Hello, {name}";
        }
        
        // 若需要其它參數, 補在後面即可
        public static string HelloWithAge(this string name, int age)
        {
            return $"Hello, {name}, My Age is {age}";
        }
    }
}
```
使用方法如下:
```csharp
// 引用包含擴充方法的命名空間
using MyExtensions;

// ...

var name = "John";
ConsoleWriteLine(name.Hello());
ConsoleWriteLine(name.HelloWithAge(30))
```
>Hello, John
<br/>Hello, John, My Age is 30

<br/>使用上看起來就像該類別的本身的方法一樣
<br/>其實這是 compiler 在編譯期間幫你轉成以下寫法

```csharp

namespace MyExtensions
{
    public static class UtilityExtensions
    {
        // 拿掉 this
        public static string Hello(string name)
        {
            return $"Hello, {name}";
        }        

        // 拿掉 this
        public static string HelloWithAge(string name, int age)
        {
            return $"Hello, {name}, My Age is {age}";
        }
    }
}
```

```csharp
// 引用包含擴充方法的命名空間
using MyExtensions;

// ...

var name = "John";
// 把 string 類別移到第一個參數
ConsoleWriteLine(UtilityExtensions.Hello(name));
ConsoleWriteLine(UtilityExtensions.Hello(name, 30));
```