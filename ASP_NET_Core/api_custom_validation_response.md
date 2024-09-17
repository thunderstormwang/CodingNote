# [.Net Core]ActionFilter 應用 - 自訂 Api 模型驗證回傳結構

使用平台: .Net Core 8.0

---

參數 Model:
```csharp
public class Person
{
    [Required]
    public string FirstName { get; set; }

    [Required]
    public string LastName { get; set; }
}
```
  
如果指定 controller 為 `ApiController`，預設會幫你開啓驗證  

```csharp
[Route("api/[controller]")]
[ApiController]
public class RigisterController : Controller
{
    // ...
}
```

如果傳入的 model 有錯，就會回傳 HttpStatusCode: 400 和下列結構
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "00-4072dba34d846ecd5ddf23fb8a376dca-ea9b8c98980caab5-00",
  "errors": {
    "firstName": [
      "The firstName field is required."
    ],
    "lastName": [
      "The lastName field is required."
    ]
  }
}
```

這結構可能對使用 api 的人不是那麼友善，我們會希望 api 處理成功或失敗都回傳相同的結構，那麼我們可以自訂驗證回傳的結構  
那麼有下列方法：  

## 方法一，自訂 ActionFilter

自訂 api 回傳 Model
```csharp
public class MyApiResponse<T>
{
    public string Code { get; set; }

    public T Data { get; set; }
}
```

然後依你需求，寫同步版本或非同步版本的 ActionFilter  

同步版本，你的 ActionFilter 得繼承 `IActionFilter`
```csharp
public class ValidateModelFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (context.ModelState.IsValid)
        {
            return;
        }

        var errorMessages = context.ModelState.Values.SelectMany(v => v.Errors.Select(e => e.ErrorMessage));
        var failResponse = ApiResponseVo<object>.CreateFailure(errorMessages.ToList());
        context.Result = new OkObjectResult(failResponse);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
    }
}
```

非同步版本，你的 ActionFilter 得繼承 `IAsyncActionFilter`
```csharp
public class ValidateModelFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        if (context.ModelState.IsValid)
        {
            await next();
            return;
        }

        var errorMessages = context.ModelState.Values.SelectMany(v => v.Errors.Select(e => e.ErrorMessage));
        var failResponse = ApiResponseVo<object>.CreateFailure(errorMessages.ToList());
        context.Result = new OkObjectResult(failResponse);
    }
}
```

然後關閉預設開啓的模型驗證，
再依你需求掛在 Action 或 Controller 或 Global 上面，此例為掛在 Global 上
```csharp
builder.Services.AddControllers(opt =>
    {
        // 關閉預設開啓的模型驗證
        opt.Filters.Add<ValidateModelFilter>();
   })
   .ConfigureApiBehaviorOptions(opt => 
   opt.SuppressModelStateInvalidFilter = true);

builder.Services.AddControllers(opt =>
    {
        // 將自訂 ActionFilter 註冊為 Global
        opt.Filters.Add<ValidateModelFilter>();
   })
   .ConfigureApiBehaviorOptions(opt => 
    {
        // 關閉預設開啓的模型驗證
        opt.SuppressModelStateInvalidFilter = true;
    });
```

<br/>然後依你需求掛在 Action 或 Controller 或 Global 上面

那麼回傳結構就會變成如下
```json
{
  "code": "9999",
  "data": [
    "The LastName field is required.",
    "The FirstName field is required."
  ]
}
```

---

## 方法二、自訂 InvalidModelStateResponseFactory

自訂 api 回傳 Model
```csharp
public class MyApiResponse<T>
{
    public string Code { get; set; }

    public T Data { get; set; }
}
```

然後在 program.cs 提供自己的 InvalidModelStateResponseFactory  
```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(opt =>
    {
        opt.InvalidModelStateResponseFactory = actionContext =>
        {
            var errorMessages = actionContext.ModelState.Values.SelectMany(v => v.Errors.Select(e => e.ErrorMessage));
            var failResponse = ApiResponseVo<object>.CreateFailure(errorMessages.ToList());
            return new OkObjectResult(failResponse);
        };
    });
```

或是是它抽成獨立的類別

```csharp
public class CustomModelStateValidator
{
    public static IActionResult CreateErrorResponse(ActionContext actionContext)
    {
        var errorMessages = actionContext.ModelState.Values.SelectMany(v => v.Errors.Select(e => e.ErrorMessage));
        var failResponse = ApiResponseVo<object>.CreateFailure(errorMessages.ToList());
        return new OkObjectResult(failResponse);
    }
}
```
然後在 program.cs 呼叫它
```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(opt =>
    {
        opt.InvalidModelStateResponseFactory = actionContext => CustomModelStateValidator.CreateErrorResponse(actionContext);
    });
```

那麼回傳結構就會變成如下
```json
{
  "code": "9999",
  "data": [
    "The LastName field is required.",
    "The FirstName field is required."
  ]
}
```