# [驗證] 自訂 Api 驗證回傳 Model

使用平台: .Net Core 6.0

---

Model:
```csharp
public class Person
{
    [Required]
    public string FirstName { get; set; }

    [Required]
    public string LastName { get; set; }
}
```

<br/>如果指定 controller 為 ApiController，預設會幫你開啓驗證，

如果傳入的 model 有錯，就會回傳下列結構
```json
// ReturnCode: 400

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

<br/>這結構可能對使用 api 的人不那麼友善，那麼我們可以自訂驗證回傳的 model

## 一、關閉預設開啓的驗證, 自己撰寫驗證的 ActionFilter

<br/>關閉預設開啓的驗證
```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.SuppressModelStateInvalidFilter = true;
    });
```

<br/>自訂 api 回傳 Model
```csharp
public class MyApiResponse<T>
{
    public string Code { get; set; }

    public T Data { get; set; }
}
```

<br/>撰寫 ActionFilter 繼承 IActionFilter
```csharp
public class MyModelValidationFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (context.ModelState.IsValid)
        {
            return;
        }

        var errors = context.ModelState.Values.SelectMany(t => t.Errors)
            .Select(t => t.ErrorMessage);

        var errorResponse = new MyApiResponse<IEnumerable<string>>()
        {
            Code = "9999",
            Data = errors
        };

        context.Result = new BadRequestObjectResult(errorResponse);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
    }
}
```

<br/>撰寫 ActionFilter 繼承  IAsyncActionFilter
```csharp
public class MyAsyncModelValidationFilter : IAsyncActionFilter
{
    /// <inheritdoc />
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        if (context.ModelState.IsValid)
        {
            return;
        }

        var errors = context.ModelState.Values.SelectMany(t => t.Errors)
            .Select(t => t.ErrorMessage);

        var errorResponse = new MyApiResponse<IEnumerable<string>>()
        {
            Code = "9999",
            Data = errors
        };

        context.HttpContext.Response.ContentType = "application/json";
        context.HttpContext.Response.StatusCode = (int)HttpStatusCode.BadRequest;
        await context.HttpContext.Response.WriteAsJsonAsync(errorResponse);
    }
}
```

<br/>然後依你需求掛在 Action 或 Controller 或 Global 上面

那麼回傳結構就會變成如下
```json
// ReturnCode: 400

{
  "code": "9999",
  "data": [
    "The LastName field is required.",
    "The FirstName field is required."
  ]
}
```

--- 
## 二、關閉預設開啓的驗證, 自己撰寫驗證的 ActionFilter、ExceptionFilter

---

## 三、改提供自己的 InvalidModelStateResponseFactory