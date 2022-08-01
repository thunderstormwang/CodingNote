# [C# 2.0]Using 陳述式

## using 等同 try finally

實作 IDisposable 介面
```csharp
public class ResourceObject : IDisposable
{
    public void Dispose()
    {
        Console.Writeline("Disposed!!!");
    }
}
```

<br/>使用 using
```csharp
using (ResourceObject resourceObject = new ResourceObject())
{
    Console.Writeline("ResourceObject in using statement");
}
```
>ResourceObject in using statement
<br/>Disposed!!!

<br/>compiler 會編成以下程式碼
```csharp
ResourceObject resourceObject = new ResourceObject();
try
{
    Console.Writeline("ResourceObject in try finally statement");
}
finally
{
    if (resourceObject != null)
    {
        resourceObject.Dispose();
    }
}
```
>ResourceObject in try finally statement
<br/>Disposed!!!

<br/>測試直接指派 null 的情境
```csharp
using (ResourceObject resourceObject = null )
{
    Console.Writeline("ResourceObject in using statement");
}
```
>ResourceObject in using statement

<br/>compiler 會編成以下程式碼
```csharp
ResourceObject resourceObject = null;
try
{
    Console.Writeline("ResourceObject in try finally statement");
}
finally
{
    if (resourceObject != null)
    {
        resourceObject.Dispose();
    }
}
```
>ResourceObject in try finally statement

---
## 在 using 區塊內出 exception

在 using 區塊內出例外，仍會先執行該物件的 Dispose 函式再將該例外拋出
```csharp
try
{
    using (ResourceObject resourceObject = new ResourceObject() )
    {
        Console.Writeline("ResourceObject in using statement");
        throw new Exception("Test Exception");
    }
}
catch (Exception ex)
{
    Console.Writeline(ex.ToString());
}
```
>ResourceObject in using statement
<br/>Disposed!!!
<br/>System.Exception: Test Exception