# 在 .Net Core WebApi 使用自訂的 Authentication handler 驗證 JWT

總會有些情況我們會想自訂驗證方式，例如
- jwt 不放在 header 的 Authorization 欄位，想放另一個自訂欄位
- 定義自己的驗證方式
- 設計後門，在開發期間方便測試
- 想改變驗證不過的狀態碼

定義自己的 Authentication Handler
```csharp
public class MyAuthHandler : AuthenticationHandler<MyAuthenticationSchemeOptions>
{
    /// <inheritdoc />
    public MyAuthHandler(IOptionsMonitor<MyAuthenticationSchemeOptions> options, ILoggerFactory logger, UrlEncoder encoder, ISystemClock clock) : base(options, logger, encoder, clock)
    {
    }

    /// <inheritdoc />
    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        await Task.CompletedTask;
        
        // 取得自定義的欄位
        var hello = base.Options.Hello;
        return AuthenticateResult.NoResult();
    }
}
```

定義自己的 AuthenticationSchemeOptions, 可以帶自定義的值進去
```csharp
public class MyAuthenticationSchemeOptions: AuthenticationSchemeOptions
{
    public string Hello { get; set; }
}
```

在 program.cs 指定使用自己的 handler 作驗證
```csharp
public static class AuthenticationExtension
{
    public static IServiceCollection AddMyTokenAuthentication(this IServiceCollection services, IConfiguration config)
    {
        services.AddAuthentication("MyAuth") // 預設 schema
            .AddScheme<MyAuthenticationSchemeOptions, MyAuthHandler>("MyAuth", o =>
            {
                // 傳入自定義的欄位
                o.Hello = "Hello World!";
            });
    }

    return services;
}
```

```csharp
var configuration = builder.Configuration;

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddMyTokenAuthentication(configuration);
```