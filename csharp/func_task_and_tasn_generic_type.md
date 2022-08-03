# [C#]Task 和 Task<T> 是不同

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
    public Task DoSomething())
    {
        // ...
    }
}
```

<br/>我在寫測試時，一直想著能否用同一個 func，像這樣寫個 BaseHanderTest
```csharp
public class BaseHanderTest<THandler, TResponse>
{
    protected THandler _handler;
    protected Func<Task<TResponse>> _func;
}
```

<br/>然後 Handler1Test 去繼承 BaseHanderTest
```csharp
public class Handler1Test : BaseHanderTest<Hander1, string>
{
    public Handler1Test()
    {
        _func = async () => await DoSomething();        
    }
}
```

<br/>在 Handler1Test 都沒問題，在 Handler2Test 卻一直 compiler error
```csharp
public class Handler2Test : BaseHanderTest<Handler2, Task>
{
    public Handler2Test()
    {
        // compiler error
        _func = async () => await DoSomething();        
    }
}
```
>[CS4010] 無法將非同步 Lambda 運算式 轉換成委派類型 'Task\<Task>'。非同步 Lambda 運算式 可能會傳回 void、Task 或 Task\<T>，而這些都無法轉換成 'Task\<Task>'。

<br/>這裡我卡了很久，一直認為可以用同一個 func 解決，最後才發現因為在 BaseHanderTest 的 func 是 ``` Func<Task<TResponse>>  ```。
<br/>如果 TResponse 代入 string，就是``` Func<Task<string>>  ``` 這符合 ``` public Task<string> DoSomething() ``` 的簽章。
<br/>但如果 TResponse 代入 Task，就是``` Func<Task<Task>>  ``` 這不符合 ``` public Task DoSomething() ``` 的簽章。
<br/>所以我怎麼試都試不出來...。其實錯誤訊息給得很清楚了，就是自己資質太差看不懂。

## 解法 1.
拆成不同的 BaseHandlerTest
```csharp
public class BaseHander1Test<THandler, TResponse>
{
    protected THandler _handler;
    protected Func<Task<TResponse>> _func;
}

public class Handler1Test : BaseHander1Test<Hander1, string>
{
    public Handler1Test()
    {
        _func = async () => await DoSomething();        
    }
}
```
```csharp
public class BaseHander2Test<THandler>
{    
    protected Func<Task> _func;
}

public class Handler2Test : BaseHanderTest<Handler2>
{
    public Handler2Test()
    {
        _func = async () => await DoSomething();        
    }
}
```

## 解法 2.
改變 func 的宣告，如此就可以用同一個 func，雖然編得過，但感覺怪怪的...
<br/>所以我偏好第二種解法
```csharp
public class BaseHanderTest<THandler, TResponse>
{
    protected THandler _handler;
    protected Func<TResponse> _func;
}
```
```csharp
public class Handler1Test : BaseHanderTest<Hander1, Task<string>>
{
    public Handler1Test()
    {
        _func = async () => await DoSomething();        
    }
}
```
```csharp
public class Handler2Test : BaseHanderTest<Handler2, Task>
{
    public Handler2Test()
    {
        // compiler error
        _func = async () => await DoSomething();        
    }
}
```