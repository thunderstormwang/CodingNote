# Thread.Sleep() 和 Task.Dealy() 的差別

先記到這樣，其實還有其它眉眉角角...

---

Thread.Sleep()
- 讓目前執行緒進入睡眠，阻擋目前的執行緒

<br/>Task.Dealy()
- 生出另一條執行緒去等候，不會阻擋目前的執行緒

<br/>Thread.Sleep() 範例
```csharp
public void ThreadSleepSample()
{
    var sw = new Stopwatch();
    sw.Start();
    Console.WriteLine("sync: starting");
    Thread.Sleep(5 * 1000);
    Console.WriteLine($"sync: running for {sw.Elapsed.TotalSeconds} seconds");
    Console.WriteLine("sync: done");
}
```
執行結果
>sync: starting
<br/>sync: running for 5.0048353 seconds
<br/>sync: done

<br/>Task.Dealy() 範例
```csharp
public void ThreadSleepSample()
{
    var sw = new Stopwatch();
    sw.Start();
    Console.WriteLine("sync: starting");
    var delay = Task.Delay(5* 1000);
    Console.WriteLine($"sync: running for {sw.Elapsed.TotalSeconds} seconds");
    await delay;
    Console.WriteLine($"sync: running for {sw.Elapsed.TotalSeconds} seconds");
    Console.WriteLine("sync: done");
}
```
執行結果，在 await 前並沒被阻擋，在 await 之後就真的等了 5 秒
>sync: starting
<br/>sync: running for 0.006105 seconds
<br/>sync: running for 5.009821 seconds
<br/>sync: done