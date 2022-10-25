# 在 Controller 的 Constructor 取得 HttpContext

使用平台: .Net Core 6.0

---

在 Controller 的 Constructor 無法取得 HttpContext，會得到 null
```csharp
[ApiController]
[Route("[controller]")]
public class DemoController : Controller
{
    public DemoController()
    {
        // null
        var context = HttpContext;
    }
}
```

有查到[原因](https://stackoverflow.com/questions/3236273/null-controllercontext-in-my-custom-controller-inheriting-from-basecontroller)

解法有：

一、在 Constructor 注入 IHttpContextAccessor

```csharp
[ApiController]
[Route("[controller]")]
public class DemoController : Controller
{
    public DemoController(IHttpContextAccessor accessor)
    {
        var context = accessor.HttpContext;
    }
}
```

<br/>在 program.cs 加上

```csharp
builder.Services.AddHttpContextAccessor();
```

<br/>
二、改在 OnActionExecuting 取得 HttpContext

```csharp
[ApiController]
[Route("[controller]")]
public class DemoController : Controller
{
    public override void OnActionExecuting(ActionExecutingContext ctx) 
    {
        var context = HttpContext;
    }
}
```