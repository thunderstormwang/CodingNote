# [套件] Polly

在官方文件作了這樣的介紹:
<br>Polly is a .NET resilience and transient-fault-handling library that allows developers to express policies such as Retry, Circuit Breaker, Timeout, Bulkhead Isolation, Rate-limiting and Fallback in a fluent and thread-safe manner.
<br/>Polly 是 .Net 開發用的瞬間故障處理的函式庫，它以流暢和線程安全的方式提提供(Retry)、斷路(Circuit-breaker)、超時(Timeout)、隔離(Bulkhead Isolation)、限流(Rate-limiting)、回退(Fallback) 等機制

<br/>先以其中一個用法重試做為介紹吧!!
<br/>平常我們呼叫第三方服務，為了避免網路問題，可以這樣針對呼叫動作加入重試機制

```csharp
private static int retryCount = 3;
private static void doRetry(int second, Action action)
{
    while (true)
    {
        try
        {
            action(); // do something
            break; 
        }
        catch
        {
            if (--retryCount == 0)
                throw;
            Thread.Sleep(1000 * second);
        }
    }
}
```

底下用範例表示搭配 polly 後，程式碼會變怎樣  

```csharp
var retryPolicy = Policy
    // 故障處理 : 要處理什麼例外
    .Handle<Exception>()
    // 故障處理 : 要處理什麼回傳值
    .OrResult<int>(result => result < 110)
    // 重試策略 : 異常發生時要進行的重試次數及重試機制
    .WaitAndRetryAsync(5, sleepDurationProvider: _ => TimeSpan.FromSeconds(3),
        (res, retrySpan, retryCount, _) =>
        {
            Console.WriteLine(
                $"result: {res.Result}, 例外: {res.Exception?.Message}, 重試區間: {retrySpan.TotalSeconds} sec, 重試數: {retryCount}");
        });

// 實際要做的工作
var result = await retryPolicy.ExecuteAsync(async () => await Worker.DoSomething());

Console.WriteLine($"最後 result: {result}");
```

Worker 類別  
```csharp
public class Worker
{
    private static int _count = 100;

    public static Task<int> DoSomething()
    {
        if (_count == 103)
        {
            _count++;
            throw new NotImplementedException();
        }

        _count++;
        return Task.FromResult(_count);
    }
}
```

>result: 101, 例外: , 重試區間: 3 sec, 重試數: 1
<br/>result: 102, 例外: , 重試區間: 3 sec, 重試數: 2
<br/>result: 103, 例外: , 重試區間: 3 sec, 重試數: 3
<br/>result: 0, 例外: The method or operation is not implemented., 重試區間: 3 <br/>sec, 重試數: 4
<br/>result: 105, 例外: , 重試區間: 3 sec, 重試數: 5
<br/>最後 result: 106

對我來說，就是在自己的程式碼看不到迴圈的使用(但實際上還是有迴圈)  
將重試的邏輯和實際處理的邏輯分開，讓程式碼更容易閱讀  

借用官方的圖講解他們的重試功能
![How Polly Retry works](!/../imgs/how_polly_retry_work.png)

## PolicyWrap
官方文件的解說挺詳細的，policy 之中可以再包另一個 policy

```csharp
var policyWrap = fallback.Wrap(cache).Wrap(retry).Wrap(breaker).Wrap(bulkhead).Wrap(timeout);
// or (functionally equivalent)
var policyWrap = fallback.Wrap(cache.Wrap(retry.Wrap(breaker.Wrap(bulkhead.Wrap(timeout)))));
```
上述程式碼的效果如同下圖
![PolicyWrap](!/../imgs/polly_wrap.png)

## Polly 搭配 .NET Core 的 DI
官方文件的解說挺詳細的，這樣就不再贅述

## 參考
[[NETCore] 使用 Polly 實現重試 (Retry) 策略](https://marcus116.blogspot.com/2019/06/netcore-polly-retry.html)