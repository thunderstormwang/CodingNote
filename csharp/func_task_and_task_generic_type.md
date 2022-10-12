# [C#]Task 和 Task\<T> 是不同

現在公司專案，大多分成兩種 handler

<br/>有回傳值的
```csharp
public class Handler1
{
    public Task<string> DoSomething()
    {
        // ...
    }
}
```

<br/>沒有回傳值的
```csharp
public class Handler2
{
    public Task DoSomething()
    {
        // ...
    }
}
```

<br/>我在寫測試時，一直想著能否用同一個 func，像這樣寫個泛型的基底類別 BaseSandbox
```csharp
public class BaseSandbox<THandler, TResponse>
{
    protected THandler _handler;
    protected Func<Task<TResponse>> _func;
    
    public bool TriggerHandler(TResponse expected)
    {
        var actual = _func.Invoke().ConfigureAwait(false).GetAwaiter().GetResult();
        return expected.Equals(actual);
    }
    
    public void TriggerHandler()
    {
        _func.Invoke().ConfigureAwait(false).GetAwaiter().GetResult();
    }
}
```

<br/>然後個別繼承 BaseSandbox 去寫測試

以下是 Handler1 的測試，可以編譯過
```csharp
// 繼承基底類別, 固定泛型的型別
public class Handler1Sandbox : BaseSandbox<Handler1, string>
{
    public Handler1Sandbox()
    {
        _func = async () => await _handler.DoSomething();
    }
}
```
```csharp
public class Handler1Test
{
    public void Test() =>
        new Handler1Sandbox()
            .TriggerHandler("expected");
}
```

<br/>以下是 Handler2 的測試，我想說 Void 對應到的應該是 Task 沒錯，可是卻得到 compiler error
```csharp
// 繼承基底類別, 固定泛型的型別
public class Handler2Sandbox : BaseSandbox<Handler2, Task>
{
    public Handler2Sandbox()
    {
        // compiler error
        _func = async () => await _handler.DoSomething();
    }
}
```
```csharp
public class Handler2Test
{
    public void Test() =>
        new Handler2Sandbox()
            .TriggerHandler();
}
```
>[CS4010] 無法將非同步 Lambda 運算式 轉換成委派類型 'Task\<Task>'。非同步 Lambda 運算式 可能會傳回 void、Task 或 Task\<T>，而這些都無法轉換成 'Task\<Task>'。

<br/>這裡我卡了很久，一直認為可以用同一個 func 解決，最後才發現因為在 BaseHanderTest 的 func 是 ``` Func<Task<TResponse>>  ```。

如果 TResponse 代入 string，就是 ``` Func<Task<string>>  ``` 這符合 ``` public Task<string> DoSomething() ``` 的簽章。

但如果 TResponse 代入 Task，就是 ``` Func<Task<Task>>  ``` 這不符合 ``` public Task DoSomething() ``` 的簽章。函式要求回傳 ``` Task  ``` 但 Func 是回傳 ``` Task<Task>  ```。

所以我怎麼試都試不出來...。其實錯誤訊息給得很清楚了，就是自己資質太差看不懂。

## 解法
認清到 Task 和 Task\<T> 是不一樣的之後，就還是拆成不同的基底類別

適用於回傳 Task\<T>
```csharp
public class BaseSandboxGenericTask<THandler, TResponse>
{
    protected THandler _handler;
    protected Func<Task<TResponse>> _func;
    
    public bool TriggerHandler(TResponse expected)
    {
        var actual = _func.Invoke().ConfigureAwait(false).GetAwaiter().GetResult();
        return expected.Equals(actual);
    }
}
```
```csharp
public class Handler1Sandbox : BaseSandboxGenericTask<Handler1, string>
{
    /// <inheritdoc />
    public Handler1Sandbox()
    {
        _func = async () => await _handler.DoSomething();
    }
}
```
```csharp
public class Handler1Test
{
    public void Test() =>
        new Handler1Sandbox()
            .TriggerHandler("expected");
}
```

<br/>適用於回傳 Task
```csharp
public class BaseSandboxTask<THandler>
{    
    protected THandler _handler;
    protected Func<Task> _func;
    
    public void TriggerHandler()
    {
        _func.Invoke().ConfigureAwait(false).GetAwaiter().GetResult();
    }
}
```
```csharp
public class Handler2Sandbox : BaseSandboxTask<Handler2>
{
    /// <inheritdoc />
    public Handler2Sandbox()
    {
        _func = async () => await _handler.DoSomething();
    }
}
```
```csharp
public class Handler2Test
{
    public void Test() =>
        new Handler2Sandbox()
            .TriggerHandler();
}
```