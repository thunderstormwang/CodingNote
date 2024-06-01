# [套件] HttpClient

HttpClient 在 .net framework 4.5 以後推出，感覺是推出來取代 WebClient，事實上在 MSDN 的 WebClient 的介紹也不推薦你再繼續使用

<br/>HttpClient 有實作 IDispose 介面，而我們長久寫 C# 的習慣，能用 using 包起來就會包起來，偏偏這個類別不適合這樣做，網路上可以找到人實測和解釋原因，大意是大量使用下會讓 port 來不及被釋放，還有 DNS 有若更新 HttpClient 仍會使用舊值，在這裡就不詳述了。

<br/>在另一篇官方文件 [Guidelines for using HttpClient](https://docs.microsoft.com/zh-tw/dotnet/fundamentals/networking/httpclient-guidelines) 也推薦使用方法

## 在 .NET Framework
每次使用時都建立的新的新例，也就是可以用 using 語法包起來

## 在 .NET Core 和 .NET 5+
+ 用 static 或 singleton
+ 與 IHttpClientFactory 搭配使用

在這裡要介紹如何在 .NET Core 如何對 IHttpClientFactory 做 DI
<br>這篇官方文章 [在 ASP.NET Core 中使用 IHttpClientFactory 發出 HTTP 要求](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/http-requests?view=aspnetcore-6.0) 介紹了四種方法，底下寫出具名用戶端

在 StartUp.cs 加入
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // other code ...

    services.AddHttpClient("Hello", c => {
        c.BaseAddress = "example.com";
    });
}
```
```csharp
public class HelloService
{
    private readonly IHttpClientFactory _httpClientFactory;

    public HelloService(IHttpClientFactory httpClientFactory) =>
        _httpClientFactory = httpClientFactory;

    public async Task OnGet()
    {
        var httpClient = _httpClientFactory.CreateClient("GitHub");
        
        // other code ...
    }
}
```