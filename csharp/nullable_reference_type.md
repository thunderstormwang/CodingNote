# [C# 8.0]Nullable Refence Type

使用平台: .Net Core 6.0

---

```csharp
// 在 C# 8.0 以後會出現 compiler warning
string s1 = null;
```
>  [CS8600] 正在將 Null 常值或可能的 Null 值轉換為不可為 Null 的型別。

<br/>

```csharp
// 在 C# 8.0 之前會 compiler error, 在 8.0 以後可以編繹成功
string? s2 = null; 
```


<br/>引入 nullable ref type 的用意是讓 compiler 可以幫你檢查可能發生 null reference 的地方

例如我們宣告這樣的函式, compiler 會認為參數不能為 null
```csharp
static int Length(string text)
{
    return text.Length;
}
```

<br/>所以當我們這樣呼叫時, 會跳出 compiler warning
```csharp
Length(null);
```
>[CS8625] 無法將 null 常值轉換成不可為 Null 的參考型別。


<br/>但它也只能提示 warning，無法造成 build error，這樣的做法是避免讓舊版的程式一升上去後，就要修很多地方。

如果覺得這警告很煩，也是可以關掉的，在 .csproj 檔案將 Nullable 設為 disable 或是將 Nullable 那行拿掉，那就等於關閉 nullable reference type 的功能，也就不會再跳這警告了。

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
        <LangVersion>10</LangVersion>
    </PropertyGroup>

</Project>

```

---

參考自
- [初探 C# 8 的 Nullable Reference Types](https://www.huanlintalk.com/2018/02/c-8-nullable-reference-types.html)