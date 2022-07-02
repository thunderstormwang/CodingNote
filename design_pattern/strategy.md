# Design Pattern + Strategy

+ 定義一組演算法，將每個演算法都封裝起來，並且使它們可以互換，在不影響外界的情況下個別抽換所使用的演算法。
+ 具體策略暴露的問題。
+ 跟 State 很像，只是狀態由外部決定

```mermaid
classDiagram
class StrategyContext {
    +StrategyContext _strategy
    +StrategyContext(AbstractStrategy strategy)
    +ExecuteStrategy()
}

class AbstractStrategy {
    +AbstractStrategy()
    +AlgorithmMethod()
}

class Strategy1 {
    +Strategy1()
    +AlgorithmMethod()
}

class Strategy2 {
    +Strategy2()
    +AlgorithmMethod()
}

StrategyContext "1" ..> "1" AbstractStrategy
AbstractStrategy <|-- Strategy1
AbstractStrategy <|-- Strategy2
```

<br/>策略類別，實作演算法內容
```csharp
public abstract class AbstractStrategy
{
    public abstract void AlgorithmMethod();
}

public class Strategy1 : AbstractStrategy
{
    public override void AlgorithmMethod()
    {
        Console.WriteLine("Strategy 1");
    }
}

public class Strategy2 : AbstractStrategy
{
    public override void AlgorithmMethod()
    {
        Console.WriteLine("Strategy 2");
    }
}
```

<br/>StrategyContext 類別，制定讓 Client 端呼叫的介面
```csharp
public class StrategyContext
{
    private AbstractStrategy _strategy;
    public StrategyContext(AbstractStrategy strategy)
    {
        _strategy = strategy;
    }
    public void ExecuteStrategy()
    {
        _strategy.AlgorithmMethod();
    }
}
```

<br/>Client 端程式
```csharp
// 這會讓 Client 端程式知道實作內容"Strategy1"
StrategyContext context = new StrategyContext(new Strategy1());
context.ExecuteStrategy();
```

## 主流寫法，對 Client 端隱藏策略類別

```mermaid
classDiagram
class StrategyContext {
    +StrategyContext _strategy
    +StrategyContext(AbstractStrategy strategy)
    +ExecuteStrategy()
}

class AbstractStrategy {
    +AbstractStrategy()
    +AlgorithmMethod()
}

class Strategy1 {
    +Strategy1()
    +AlgorithmMethod()
}

class Strategy2 {
    +Strategy2()
    +AlgorithmMethod()
}

StrategyContext "1" ..> "1" AbstractStrategy
AbstractStrategy <|-- Strategy1
AbstractStrategy <|-- Strategy2
```

+ AbstractStrategy
  + 定義所有演算法的共用介面，StrategyContext 透過這個介面呼叫 ConcreateStrategy (如圖中的 Strategy1 和Strategy2) 所具體實作的演算法。
  + 可以是interface 或abstract class。
+ ConcreteStrategy
  + 依據 AbstractStrategy 所定義的介面具體實作其演算法的內容。
+ StrategyContext
  + 制定讓Client 呼叫的介面
  + 持有AbstractStrategy 型別的執行個體(如圖中的 Strategy1 和Strategy2)
  + 可能會定義一個讓Strategy 可以呼叫的介面，讓 Strategy 可以存取Context 的資料。
+ Client
  + 呼叫StrategyContext，將外部狀態(條件)傳遞給 StrategyContext，若 StrategyContext 具有回傳值則取回執行結果。

<br/>策略類別，實作演算法內容
```csharp
public abstract class AbstractStrategy
{
    public abstract void AlgorithmMethod();
}

public class Strategy1 : AbstractStrategy
{
    public override void AlgorithmMethod()
    {
        Console.WriteLine("Strategy 1");
    }
}

public class Strategy2 : AbstractStrategy
{
    public override void AlgorithmMethod()
    {
        Console.WriteLine("Strategy 2");
    }
}
```

<br/>StrategyContext 類別，制定讓 Client 端呼叫的介面
```csharp
public class StrategyContext
{
    public void ExecuteStrategy(string args)
    {
        var strategy = StrategyFactory.CreateStrategy(args);
        if (strategy != null)
        {
            strategy.AlgorithmMethod();
        }
    }
}
```


<br/>StrategyFactory 類別，依外部參數決定該採取何種策略
```csharp
public class StrategyFactory
{
    public static AbstractStrategy CreateStrategy(string args)
    {
        switch (args)
        {
            case "Strategy1":
                return new Strategy1();
            case "Strategy2":
                return new Strategy2();
            default:
                return null;
        }
    }
}
```


<br/>Client 端程式
```csharp
// 與 simple factory 很像, 但 strategy 只需知道執行結果就好,
// simple factory 會需要知道物件實體
StrategyContext context = new StrategyContext();
context.ExecuteStrategy("Strategy2");
```

<br/>主流寫法(講師提供的範例 Strategy002)跟 Simple Factory 很像，講師解釋：
<br/>Strategy 對 Client 端程式來講，只需關心執行的結果，不必在意策略物件的實體是什麼
Simple Factory，在 client 來講，關心的是回傳的物件實體

<br/>Chain Of Responsibility vs State vs Strategy
<br/>Chain Of Responsibility：1)如果不只一個物件需要處理請求、或不確定要處理請求的物件數量，2)每次處理請求皆須將請求鏈重頭跑一次才知道哪些物件要處理
<br/>State：只有一個物件要處理請求，且狀態由內部決定，如果需要外部才能決定，那應該用 Strategy
<br/>Strategy：只有一個物件要處理請求，且狀態由外部決定

<br/>農夫渡河 Strategy vs State
<br/>老師解釋，這兩種他會選擇使用 State，因為下一個狀態很固定，由左到右後，下一個一定是由右到左，而且初始一定在左邊，如果是這樣，那 Client 端程式不需要關心現在在左邊還是在右邊，給子程式去記住現在在左邊或是在右邊就好了

<br/>農夫渡河 Strategy vs Chain Of Responsibility
老師解釋，這兩種會選擇 Strategy，因為此例只有一個物作需動作(由左移到右或由右移到左)