# [C#]Async/Await - Best Practices in Asynchronous Programming 筆記

MSDN 的這篇，[Async/Await - Best Practices in Asynchronous Programming](https://msdn.microsoft.com/en-us/magazine/jj991977.aspx)，敘述非同步程式的較好的實作法方法
此文寫在 2013 年，所以較新的技術，例如 .Net Core 就沒有提到是否要照著遵守

裡面提到三點
- Avoid async void
- Async all the way
- Configure context

## Avoid async void

能使用 Task 或 Task<T> 方法就使用，儘量避免使用 async void，除非它是 event handler  

async void 沒有 Task object，丟出的例外無法被抓到，如下
```csharp
private async void ThrowExceptionAsync() 
{
    throw new InvalidOperationException(); 
} 

public void AsyncVoidExceptions_CannotBeCaughtByCatch() 
{ 
    try 
    { 
        ThrowExceptionAsync(); 
    } 
    catch (Exception) 
    { 
        // The exception is never caught here! 
        throw; 
    } 
}
```

用 Task 或 Task\<T> 可以輕易地搭配 Task.WhenAny 和 Task.WhenAll，而 async void 無法輕易地做到，當它們完成工作時通知呼叫者，是可以實作自訂的 SynchronizationContext，但對於普通的應用程式來說，它是個複雜的作法。  

在語意上，用 Task 或 Task\<T> 在方法上，讀程式碼者可以容易理解這是非同步方法，用 async void 則很可能被略過此點。  

雖然 async void 有許多缺點，但很適合用在 eventhandler，因為 eventhandler 在本質上就是回傳 void。  

## Async all the way

不要混合同步方法與非同步方法，也就是，不要在同步方法裡呼叫非同步方法，也不要主動阻擋程式。當你需要呼叫非同步方法，就讓那段程式碼由下至上(或者你喜歡說是從上到下)都使用 async。  

以下程式碼在 Console 程式沒問題，但是在 GUI、ASP.Net 應用程式上則會遇到 Deadlock
```csharp
public static class DeadlockDemo 
{ 
    private static async Task DelayAsync() 
    { 
        await Task.Delay(1000); 
    } 

    // This method causes a deadlock when called in a GUI or ASP.NET context. 
    public static void Test() 
    { 
        // Start the delay. 
        var delayTask = DelayAsync(); 
        // Wait for the delay to complete. 
        delayTask.Wait();
    } 
}
```
原因是在 GUI、ASP.Net 應用程式中，執行緒有 SynchronizationContext，當一個 Task 被 await，當前執行緒的 SynchronizationContext 被記錄起來，以便當 Task 完成後，要用來繼續執行接下來的程式碼。  
所以流程是當前執行緒帶著 SynchronizationContext 走到 await 時就跳回到 DelayAsync 繼續往下執行，而遇到 Task.Wait 或 Task.Result 時(以上圖是 Task.Wait)，會停住等 Task 完成，當 Task 完成後，它預期會有個執行緒帶著先前記錄的 SynchronizationContext 過來繼續往下執行，但是它等不到，因為該執行緒已在等待狀態中。也就造成死結。  

當你在同步方法呼叫非同步方法，你會需要呼叫 Task.Wait 或 Task.Result，這會讓丟出的例外被包在 AggregateException，增加例外處理的麻煩。  

裡面提到唯一的例外是 Console 程式，否過 C# 7.0 以後也支援 async Main 方法了，就沒有這個例外了。  

總結一下你該採用的非同步程式的寫法，如下表，The "Async Way" of Doing Things

| To Do This ...                              | Instead of This          | Use This           |
|---------------------------------------------|--------------------------|--------------------|
| Retrieve the result of a background task    | Task.Wait or Task.Result | await              |
| Wait for any task to complete               | Task.WaitAny             | await Task.WhenAny |
| Retrieve the results of multiple tasks task | Task.WaitAll             | await Task.WhenAll |
| Wait a period of time                       | Thread.Sleep             | await Task.Delay   |

## Configure context

儘量使用 ConfigureWait，同時也可避免 Deadlock，唯一的例外是若 Task 完成後的程式碼需要 context 去執行。  

如果 await 後面的程式碼不需要
- UI thread 去更新畫面元件(Winform、WPF)
- HttpContext.Current 或回覆 ASP .NET response 包含在 action 回傳結果(.NET MVC、.NET WebForm)

可以這樣寫
```csharp
await DoSomethingAsync().ConfigureAwait(false);
```
告訴電腦 await 之後的程式碼不用 context 也可執行(反之，若需要 context 就不可這樣做)，那電腦會從 thread pool 選出一條執行緒出來執行 await 之後的程式碼，達成平行處理的效果。

例如以下例子
```csharp
private async void button1_Click(object sender, EventArgs e) 
{ 
    button1.Enabled = false; 

    try 
    { 
        // Can't use ConfigureAwait here ... 
        await Task.Delay(1000); 
    } 
    finally 
    { 
        // Because we need the context here. 
        button1.Enabled = true; 
    } 
}
```

而每個非同步方法有自己的 context，以下例子是可以的
```csharp
private async Task HandleClickAsync() 
{ 
    // Can use ConfigureAwait here. 
    await Task.Delay(1000).ConfigureAwait(continueOnCapturedContext: false); 
} 

private async void button1_Click(object sender, EventArgs e) 
{ 
    button1.Enabled = false; 
    try 
    { 
        // Can't use ConfigureAwait here. 
        await HandleClickAsync(); 
    } 
    finally 
    { 
        // We are back on the original context for this method. 
        button1.Enabled = true; 
    } 
}
```

## 結論

Summary of Asynchronous Programming Guidelines


| Name              | Description                                       | Exceptions                   |
|-------------------|---------------------------------------------------|------------------------------|
| Avoid async void  | Prefer async Task methods over async void methods | Event handlers               |
| Async all the way | Don't mix blocking and async code                 | Console main method          |
| Configure context | Use ConfigureAwait(false) when you can            | Methods that require context |

Solutions to Common Async Problems


| Problems                                        | Solution                                                                  |
|-------------------------------------------------|---------------------------------------------------------------------------|
| Create a task to execute code                   | Task.Run or TaskFactory.StartNew (not the Task constructor or Task.Start) |
| Create a task wrapper for an operation or event | TaskFactory.FromAsync or TaskCompletionSource\<T\>                        |
| Support cancellation                            | CancellationTokenSource and CancellationToken                             |
| Report progress                                 | IProgress\<T\> and Progress\<T\>                                          |
| Handle streams of data                          | TPL Dataflow or Reactive Extensions                                       |
| Synchronize access to a shared resource         | SemaphoreSlim                                                             |
| Asynchronously initialize a resource            | AsyncLazy\<T\>                                                            |
| Async-ready producer/consumer structures        | TPL Dataflow or AsyncCollection\<T\>                                      |
