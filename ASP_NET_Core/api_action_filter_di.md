# [.Net Core]Action Filter 應用 - 使用 DI 注入參數

- [\[.Net Core\]Action Filter 應用 - 使用 DI 注入參數](#net-coreaction-filter-應用---使用-di-注入參數)
  - [當 Action Filter 的建構式只需要常數做參數](#當-action-filter-的建構式只需要常數做參數)
  - [當 Action Filter 的建構式需要的參數都可來自 DI](#當-action-filter-的建構式需要的參數都可來自-di)
  - [當 Action Filter 的建構式需要的參數只有部份可來自 DI](#當-action-filter-的建構式需要的參數只有部份可來自-di)


使用平台: .Net Core 8.0  

## 當 Action Filter 的建構式只需要常數做參數

用法跟以前一樣，繼承 `ActionFilterAttribute`  
```csharp
public class MyAttribute : ActionFilterAttribute
{
    private string _str1;
    private string _str2;

    public MyAttribute(string str1, string str2)
    {
        _str1 = str1;
        _str2 = str2;
    }
}
```

然後掛在 Controller 或 Action 上即可  

```csharp
[Route("api/[controller]")]
[ApiController]
[MyTypeActionAttritube("我來自 Controller 字串1", "我來自 Controller 字串2")]
public class DemoController : Controller
{
    [HttpGet("Get")]
    [MyAttribute("我來自 Action 字串1", "我來自 Action 字串2")]
    public IActionResult Get()
    {
        return Ok($"hello world!!");
    }
}
```

---

## 當 Action Filter 的建構式需要的參數都可來自 DI  

當建構式需要參數時，以此例來需要 `ILogger<MyAttribute>`  

```csharp
public class MyAttribute : ActionFilterAttribute
{
    private ILogger<MyAttribute> _logger;

    public MyAttribute(ILogger<MyAttribute> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation($"{nameof(MyAttribute)} in");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        _logger.LogInformation($"{nameof(MyAttribute)} out");
    }
}
```

像以前一樣掛在 Controller 或 Action 上會 build error

```csharp
[Route("api/[controller]")]
[ApiController]
[MyAttribute]
public class DemoController : Controller
{
    [HttpGet("Get")]
    public IActionResult Get()
    {
        return Ok($"hello world!!");
    }
}
```

IDE 給的錯誤訊息如下  
>[CS7036] 未提供任何可對應到 'MyAttribute.MyAttribute(ILogger<MyAttribute>)' 之必要參數 'logger' 的引數

如果建構式需要的參數都可來來自 DI，那我們可以透過 `ServiceFilter` 來注入參數  

在 program.cs 需要向 DI 注冊 `MyAttribute`  

```csharp
builder.Services.AddScoped<MyAttribute>();
```

在 Controller 或 Action 上使用 `ServiceFilter` 來注入參數  

```csharp
[Route("api/[controller]")]
[ApiController]
[ServiceFilter<MyAttribute>]
public class DemoController : Controller
{
    [HttpGet("Get")]
    public IActionResult Get()
    {
        return Ok($"hello world!!");
    }
}
```

即可正常運作。  
且以此例來說，`MyAttribute` 可以不用繼承 `ActionFilterAttribute`，改繼承 `ActionFilter` 也可以

---

## 當 Action Filter 的建構式需要的參數只有部份可來自 DI  

如果建構式需要的參數只有部份可來自 DI，其它參數是常數  
例如以下的 Action Filter  
`ILogger<MyActionFilter>` 是 DI 注入的，`str1` 和 `str2` 是常數

```csharp
public class MyActionFilter : IActionFilter
{
    private ILogger<MyActionFilter> _logger;
    private readonly string _str1;
    public readonly string _str2;

    public MyActionFilter(ILogger<MyActionFilter> logger, string str1, string str2)
    {
        _str2 = str2;
        _logger = logger;
        _str1 = str1;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation($"{nameof(MyActionFilter)} {_str1} {_str2} in");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        _logger.LogInformation($"{nameof(MyActionFilter)} {_str1} {_str2} out");
    }
}
```

我們可以使用 `TypeFilter`，它會自動解析哪些參數可來自 DI，其它參數則從 `Arguments` 傳入  

宣告另一個 Attribute 來繼承 `TypeFilterAttribute`，並指定 `MyActionFilter` 為參數  
從建講式取得參數 `str1`, `str2`，並指定給 `Arguments`  

```csharp
public class MyTypeAttritube : TypeFilterAttribute<MyTypeActionFilter>
{
    public MyTypeAttritube(string str1, string str2)
    {
        Arguments = new object[] { str1, str2 };
    }
}
```

在 controller 或 action 上使用 `MyTypeAttritube` 來注入參數  

```csharp
[Route("api/[controller]")]
[ApiController]
[MyTypeAttritube("Controller字串1", "Controller字串2")]
public class DemoController : Controller
{
    [HttpGet("Get")]
    [MyTypeAttritube("Action字串1", "Action字串2")]
    public IActionResult Get()
    {
        return Ok($"hello world!!");
    }
}
```

在 program.cs 則不用再注冊 `MyActionFilter` 或 `MyTypeAttritube`  