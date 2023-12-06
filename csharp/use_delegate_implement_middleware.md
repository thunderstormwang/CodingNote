# [C#]用 Delegate 實現 Middleware

宣告 Delegate，和參數和回傳值都是該 Delegate 的函式  
加入印出訊息，更容易了解執行順序

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

這段很難懂
```csharp
public RequestDelegate BazMiddleware(RequestDelegate next) =>
    context =>
    {
        Console.WriteLine($"Before BazMiddleware, context: {context}");
        var temp =  next(context + 1);
        Console.WriteLine($"After BazMiddleware, context: {context}");
        return temp;
    };
```

可以看作是
```csharp
    public RequestDelegate BarMiddleware(RequestDelegate next)
    {
        return context =>
        {
            Console.WriteLine($"Before BarMiddleware, context: {context}");
            var temp = next(context + 1);
            Console.WriteLine($"After BarMiddleware, context: {context}");
            return temp;
        };
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