# [C#]用 stopwatch 測試效能，去掉最大最小值後再算平均

```csharp
var repetitions = 1000;
var executeTimes = new long[repetitions]
for (var i = 0; i < repetitions; i++)
{
    var stopwatch = Stopwatch.StartNew();
    
    // TODO: 你想測試的程式碼片段
    
    stopwatch.Start();
    executeTimes[i] = stopwatch.ElapsedMilliseconds;
}

Array.Sort(executeTimes)
// 去掉最快的 5% 和最慢的 5%, 取中間的 90% 來計算平均值
var totalTime = 0L;
var discardCount = (int)Math.Round(repetitions * 0.05);
var count = repetitions - discardCount * 2;
for (var i = discardCount; i < count; i++)
{
    totalTime += executeTimes[i];
}

var averageTime = (double)totalTime / count;
Console.WriteLine($"Average time: {averageTime} ms");
```