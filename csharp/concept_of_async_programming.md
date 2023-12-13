# [C#]Asynchronous Programing concept 非同步程式的觀念

試想，你的程式有兩個工作要做
- 工作 A，總共要跑 20 秒鐘，開頭跑 5 秒，中間的 10 秒在等待(I/O 或遠端伺服器)回應，結尾再跑 5 秒
- 工作 B，不間斷地跑 5 秒鐘

按照同步的做法，程式就是先做 A 再做 B，或是先 B 再 A ，總共都要 25 杪。  
但以人的角度的來看，我們會選擇先做 A，但 A 做到需要等待時，留下一個標記記錄做到哪裡，就離開去做 B，等到 A 完成後，就從剛才的標記繼續做，如此就只需要 20 秒。非同步即是這樣的概念

- <b>非同步不會讓你的程式跑更快，它只能讓你的程式跑得更有效率。</b>以上述例子來說，A、B的個別完成時間並沒縮短，我們只是有效地利用了在 A 中間等待時間去做 B，在 20 秒鐘完成 A + B
- <b>非同步不是多線程。</b>以上述例子來說，我們仍只用一個線程去做 A、B，實際上就算分成兩個線程分別做，也不會比較快，一樣需要 20 秒鐘完成 A + B
- 非同步在 I/O bound 的工作很有效果，在 CPU bound 的工作則沒什麼效果

## 語法

async
- 宣告此方法為非同步，可在方法內使用 await 關鍵字
- 該方法只能有三種回傳值：void、Task、Task<T>
- 若方法裡面不包含 await，不會有 compiler error，但會有 compiler warning
- 進入 async 方法不代表程式就分2條路執行了

await
- 程式在 async 方法內也是以同步方式執行，直到遇到 await 才停住等工作完成(並不會封鎖當前執行緒，只讓該執行緒等待)，同時控制權返回 async 方法的呼叫端

## Demo Code

### 模擬同步的處理方式
用 Thread.Sleep 模擬一個需等待的動作，例如等待 I/O。
```csharp
public class ThreadClass
{
    public string DoSomething()
    {
        Thread.Sleep(3 * 1000);
        return "Finished";
    }
}
```

然後呼叫它
```csharp
var stopWatch = new Stopwatch();
stopWatch.Start();
Console.WriteLine("synchonous start");
var result = new ThreadClass().DoSomething();

for (var i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}

Console.WriteLine($"for loop done, {stopWatch.Elapsed.TotalSeconds}");
Console.WriteLine($"result: {result}");
Console.WriteLine("synchonous end");        
```

執行結果，可以看出 foreach 是 3 秒之後才做完，代表當前執行緒被阻擋，等 DoSomething(模擬的 I/O) 作完才能繼續

>synchonous start  
0  
1  
2  
3  
4  
5  
6  
7  
8  
9  
for loop done, 3.0209091  
result: Finished  
synchonous end  

### 模擬非同步的處理方式

用 Task.Delay 模擬一個需等待的動作，例如等待 I/O。

```csharp
public class AsyncClass
{
    public string DoSomethingAsync()
    {
        Task.Delay(3 * 1000);
        return "Finished";
    }
}
```

然後呼叫它
```csharp
var stopWatch = new Stopwatch();
stopWatch.Start();
Console.WriteLine("asynchonous start");
var result = new AsyncClass().DoSomethingAsync();

for (var i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}

Console.WriteLine($"for loop done, {stopWatch.Elapsed.TotalSeconds}");
Console.WriteLine($"result: {result}");
Console.WriteLine("asynchonous end");        
```

執行結果，可以看出執行緒呼叫 DoSomethingAsync(模擬的 I/O) 後，沒被阻擋就接著做完 foreach，接著才是等待 DoSomethingAsync 的結果。
這是因為程式碰到 await 即跳回呼叫端，繼續往下執行，直到碰到 Task.Result 才會停下等待非同步方法的執行結果

>asynchonous start  
0  
1  
2  
3  
4  
5  
6  
7  
8  
9  
for loop done, 0.0142772  
result: Finished  
asynchonous end
