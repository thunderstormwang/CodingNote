# [.Net Core]Exception Filter 應用

使用平台: .Net Core 8.0  

根據 [MSDN(Filters in ASP.NET Core)](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-8.0#exception-filters)，ExceptionFilter 
- 可以補捉到 Razor Page or controller creation, model binding, action filters, or action methods.
- 在 resource filters, result filters, or MVC result execution 則無法補捉到

撰寫自訂的 ExceptionFilter，記錄 log，且包裝錯誤訊後再回傳  

```csharp
public class ExceptionFilter : IExceptionFilter
{
    private readonly ILogger<ExceptionFilter> _logger;

    public ExceptionFilter(ILogger<ExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        var exception = context.Exception;
        _logger.LogError(exception.ToString());

        var failResponse = ApiResponseVo<object>.CreateFailure(new List<string>() { exception.Message });
        // 代表例外已經處理過了, 避免此例再向外擴散
        context.ExceptionHandled = true;
        context.Result = new JsonResult(failResponse)
        {
            StatusCode = 200
        };
    }
}
```

宣告自訂的回傳類別

```csharp
public class ApiResponseVo<T>
{
    public bool IsSuccess { get; set; }
    
    public List<string> ErrorMessages { get; set; }
    
    public T Data { get; set; }

    public static ApiResponseVo<T> CreateSuccess(T data)
    {
        return new ApiResponseVo<T>
        {
            IsSuccess = true,
            Data = data
        };
    }

    public static ApiResponseVo<T> CreateFailure(List<string> errorMessages)
    {
        return new ApiResponseVo<T>
        {
            IsSuccess = false,
            ErrorMessages = errorMessages
        };
    }
}
```

在 progrma.cs 將此 ExceptionFilter 註冊為全域

```csharp
builder.Services.AddControllers(opt =>
    {
        opt.Filters.Add<ExceptionFilter>();
    });
```