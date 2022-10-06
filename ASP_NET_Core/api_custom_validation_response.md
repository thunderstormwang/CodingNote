# [驗證] 自訂 Api 驗證回傳 Model

使用平台: .Net Core 6.0

---

Model:
```csharp
public class Person
{
    [Required]
    public string Name { get; set; }

    [Required]
    public string Email { get; set; }
}
```

<br/>如果指定 controller 為 ApiController，預設會幫你開啓驗證，
<br/>如果傳入的 model 有錯，就會回傳下列結構
```json
// ReturnCode: 400

{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "00-4072dba34d846ecd5ddf23fb8a376dca-ea9b8c98980caab5-00",
  "errors": {
    "Name": [
      "The Name field is required."
    ]
  }
}
```

<br/>這結構可能對使用 api 的人不那麼友善，那麼我們可以自訂驗證回傳的 model

<br/>關閉預設開啓的驗證
```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.SuppressModelStateInvalidFilter = true;
    });
```

<br/>撰寫 ActionFilter
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
        var errorMessage = string.Join(", ", errors);

        context.Result = new BadRequestObjectResult(errorMessage);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
    }
}
```

<br/>那麼回傳結構就會變成如下
```
// ReturnCode: 400

The Name field is required., The Email field is required.
```