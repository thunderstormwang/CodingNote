# [C#][.Net Core]Middleware 原理和利用它記錄 api log

- [\[C#\]\[.Net Core\]Middleware 原理和利用它記錄 api log](#cnet-coremiddleware-原理和利用它記錄-api-log)
  - [如何使用](#如何使用)
    - [用 lambda 實作 Middleware](#用-lambda-實作-middleware)
    - [將 Middleware 寫成獨立的類別](#將-middleware-寫成獨立的類別)
  - [原理，用 Delgate 實現 Middleware](#原理用-delgate-實現-middleware)
  - [用 Middleware 記錄 api log](#用-middleware-記錄-api-log)
  - [ExceptionHandleMiddle](#exceptionhandlemiddle)

使用平台: .Net Core 8.0  

## 如何使用

如下圖，像 Stack，先進後出，可在每一層做加工，例如 log、驗證、壓縮、加密等等    
![Middleware](imgs/middleware.png)

### 用 lambda 實作 Middleware

用 `IWebApplication.Use` 加入 Middleware  
在 program.cs 中加入如下程式碼

```csharp
app.Use(async (context, next) =>
{
    await context.Response.WriteAsync($"Middleware 1 in{Environment.NewLine}");
    // call the next delegate/middleware in the pipeline
    await next.Invoke(context);
    await context.Response.WriteAsync($"Middleware 1 out{Environment.NewLine}");
});

app.Use(async (context, next) =>
{
    await context.Response.WriteAsync($"Middleware 2 in{Environment.NewLine}");
    // call the next delegate/middleware in the pipeline
    await next.Invoke(context);
    await context.Response.WriteAsync($"Middleware 2 out{Environment.NewLine}");
});

app.Use(async (context, next) =>
{
    await context.Response.WriteAsync($"Middleware 3 in{Environment.NewLine}");
    // call the next delegate/middleware in the pipeline
    await next.Invoke(context);
    await context.Response.WriteAsync($"Middleware 3 out{Environment.NewLine}");
});

app.Run(async context =>
{
    await context.Response.WriteAsync($"Hello World!!{Environment.NewLine}");
});

app.Run();
```

執行結果如下
>Middleware 1 in  
>Middleware 2 in  
>Middleware 3 in  
>Hello World!!  
>Middleware 3 out  
>Middleware 2 out  
>Middleware 1 out  

### 將 Middleware 寫成獨立的類別

```csharp
public class Middleware1
{
    private readonly RequestDelegate _next;

    public Middleware1(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        await context.Response.WriteAsync($"Middleware 1 in{Environment.NewLine}");
        // call the next delegate/middleware in the pipeline
        await _next.Invoke(context);
        await context.Response.WriteAsync($"Middleware 1 out{Environment.NewLine}");
    }
}
```

```csharp
public class Middleware2
{
    private readonly RequestDelegate _next;

    public Middleware2(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        await context.Response.WriteAsync($"Middleware 2 in{Environment.NewLine}");
        // call the next delegate/middleware in the pipeline
        await _next.Invoke(context);
        await context.Response.WriteAsync($"Middleware 2 out{Environment.NewLine}");
    }
}
```

```csharp
public class Middleware3
{
    private readonly RequestDelegate _next;

    public Middleware3(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        await context.Response.WriteAsync($"Middleware 3 in{Environment.NewLine}");
        // call the next delegate/middleware in the pipeline
        await _next.Invoke(context);
        await context.Response.WriteAsync($"Middleware 3 out{Environment.NewLine}");
    }
}
```

用 `IWebApplication.UseMiddleware` 加入 Middleware  
在 program.cs 中加入如下程式碼

```csharp
app.UseMiddleware<Middleware1>();
app.UseMiddleware<Middleware2>();
app.UseMiddleware<Middleware3>();

app.Run(async context =>
{
    await context.Response.WriteAsync($"Hello World!!{Environment.NewLine}");
});

app.Run();
```

執行結果如下
>Middleware 1 in  
>Middleware 2 in  
>Middleware 3 in  
>Hello World!!  
>Middleware 3 out  
>Middleware 2 out  
>Middleware 1 out 

## 原理，用 Delgate 實現 Middleware

宣告 Delegate，和參數和回傳值都是該 Delegate 的函式  
在每個函式加入印出訊息，更容易了解執行順序

```csharp
public delegate int RequestDelegate(int input);

public RequestDelegate BarMiddleware(RequestDelegate next) =>
    context =>
    {
        Console.WriteLine($"Before BarMiddleware, context: {context}");
        var temp = next(context + 1);
        Console.WriteLine($"After BarMiddleware, context: {context}");
        return temp;
    };

public RequestDelegate BazMiddleware(RequestDelegate next) =>
    context =>
    {
        Console.WriteLine($"Before BazMiddleware, context: {context}");
        var temp =  next(context + 1);
        Console.WriteLine($"After BazMiddleware, context: {context}");
        return temp;
    };

public RequestDelegate FooMiddleware(RequestDelegate next) =>
    context =>
    {
        Console.WriteLine($"Before FooMiddleware, context: {context}");
        var temp = next(context + 1);
        Console.WriteLine($"After FooMiddleware, context: {context}");
        return temp;
    };
```

中間或許不好懂，我們每個 Middleware 的參數和回傳值都是 `RequestDelegate`，而 `RequestDelegate` 定義它的參數和回傳值都是 `int`。  
拆開成如下，或許會較清楚

```csharp
public RequestDelegate AFooMiddleware(RequestDelegate next)
{
    RequestDelegate returnDel = context =>
    {
        Console.WriteLine($"FooMiddleware in, context: {context}");
        var temp = next(context + 1);
        Console.WriteLine($"FooMiddleware out, context: {context}");
        return temp;
    };

    return returnDel;
}
```

宣告 Build 函式，逐一加進 List，再反轉，再一一串連起來  

```csharp
public RequestDelegate Build()
{
    var delList = new List<Func<RequestDelegate, RequestDelegate>>();
    delList.Add(BarMiddleware);
    delList.Add(BazMiddleware);
    delList.Add(FooMiddleware);
    delList.Reverse();

    RequestDelegate next = input => input;
    foreach (var item in delList)
    {
        next = item(next);
    }

    return next;
}
```

handler 拿到的結果就是 `BarMiddleware(BazMiddleware(FooMiddleware(input)))`

```csharp
var handler = Build();
Console.WriteLine($"最後結果: {handler(5)}");
```

>Before BarMiddleware, context: 5  
Before BazMiddleware, context: 6  
Before FooMiddleware, context: 7  
After FooMiddleware, context: 7  
After BazMiddleware, context: 6  
After BarMiddleware, context: 5  
最後結果: 8  

## 用 Middleware 記錄 api log

```csharp
public class ApiLogMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ApiLogMiddleware> _logger;

    public ApiLogMiddleware(RequestDelegate next, ILogger<ApiLogMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Path.ToString().Contains("swagger"))
        {
            await _next(context);
            return;
        }

        context.Request.EnableBuffering();

        var originalRequestBody = context.Request.Body;

        string requestContent;
        using (var requestReader = new StreamReader(originalRequestBody, encoding: Encoding.UTF8, true, 1024, true))
        {
            requestContent = await requestReader.ReadToEndAsync();
        }

        originalRequestBody.Seek(0, SeekOrigin.Begin);

        _logger.LogInformation(
            $"RequestUrl: {context.Request.Path.ToString()}, QueryString: {context.Request.QueryString.ToString()}, RequestBody: {requestContent}");

        var originalResponseBody = context.Response.Body;
        
        // call the next delegate/middleware in the pipeline
        await _next(context);
        
        originalResponseBody.Seek(0, SeekOrigin.Begin);
        using (var reader = new StreamReader(originalResponseBody))
        {
            var responseContent = await reader.ReadToEndAsync();
            _logger.LogInformation($"Request Url: {context.Request.Path.ToString()}, ResponseBody: {responseContent}");
            originalResponseBody.Seek(0, SeekOrigin.Begin);
        }
    }
}
```

但是會得到如下例外，原因為 `context.Request.Body` 已經被讀取過(因為要記錄 log)，為了讓之後的處理(例如 Model Binding)可讀取就得將指針倒回去時釦出現 HttpRequestStream 並不支援 Seek 的方法  
> System.NotSupportedException: Specified method is not supported.  
> at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.Http.HttpRequestStream.Seek(Int64 offset, SeekOrigin origin)

解決方法是將 `context.Request.Body`, `context.Response.Body` 的物件替換掉，用有支援 `Stream.Seek` 的類別取代，例如 `MemorySteam`  

```csharp
public class ApiLogMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ApiLogMiddleware> _logger;

    public ApiLogMiddleware(RequestDelegate next, ILogger<ApiLogMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Path.ToString().Contains("swagger"))
        {
            await _next(context);
            return;
        }

        var replacedRequestBody = new MemoryStream();
        var originalRequestBody = context.Request.Body;

        // 替換 Request.Body
        await context.Request.Body.CopyToAsync(replacedRequestBody);
        replacedRequestBody.Seek(0, SeekOrigin.Begin);

        string requestContent;
        using (var requestReader = new StreamReader(replacedRequestBody, encoding: Encoding.UTF8, true, 1024, true))
        {
            requestContent = await requestReader.ReadToEndAsync();
        }

        replacedRequestBody.Seek(0, SeekOrigin.Begin);
        context.Request.Body = replacedRequestBody;

        _logger.LogInformation(
            $"RequestUrl: {context.Request.Path.ToString()}, QueryString: {context.Request.QueryString.ToString()}, RequestBody: {requestContent}");

        var originalResponseBody = context.Response.Body;

        using (var replacedResponseBody = new MemoryStream())
        {
            // 替換 Response.Body
            context.Response.Body = replacedResponseBody;
            
            // call the next delegate/middleware in the pipeline
            await _next(context);
            
            context.Request.Body = originalRequestBody;

            replacedResponseBody.Seek(0, SeekOrigin.Begin);
            using (var reader = new StreamReader(replacedResponseBody))
            {
                var responseContent = await reader.ReadToEndAsync();
                _logger.LogInformation($"Request Url: {context.Request.Path.ToString()}, ResponseBody: {responseContent}");
                replacedResponseBody.Seek(0, SeekOrigin.Begin);

                await replacedResponseBody.CopyToAsync(originalResponseBody);
            }
        }
    }
}
```

## ExceptionHandleMiddle

```csharp
public class ExceptionHandleMiddleware
{
    private readonly RequestDelegate _next;
    private ILogger<ExceptionHandleMiddleware> _logger;

    public ExceptionHandleMiddleware(RequestDelegate next, ILogger<ExceptionHandleMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext context)
    {
        try
        {
            // call the next delegate/middleware in the pipeline
            await _next(context);
        }
        catch (Exception ex)
        {
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            _logger.LogError($"Exception: {ex}");
            await context.Response.WriteAsync(
                $"{context.Response.StatusCode} Internal Server Error from the ExceptionHandle middleware.");
        }
    }
}
```