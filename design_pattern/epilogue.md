# 結語
- [結語](#結語)
  - [我們應該怎麼思考](#我們應該怎麼思考)
  - [反設計模式範例 - 算所得稅](#反設計模式範例---算所得稅)
    - [原本程式碼](#原本程式碼)
    - [用 Chain of Responsibility 重構](#用-chain-of-responsibility-重構)
    - [用查表法重構](#用查表法重構)
  - [反設計模式範例 - 讀卡機](#反設計模式範例---讀卡機)
    - [原本寫法](#原本寫法)
    - [採用 Vistor 模式](#採用-vistor-模式)
    - [反設計模式](#反設計模式)

## 我們應該怎麼思考
+ 為了需求而設計
+ 不要為了模式而設計
+ 衡量使用模式的代價以及它所帶來的好處
+ 何謂代價？

---

## 反設計模式範例 - 算所得稅
假設你是國稅局的資訊人員，要為計算所得稅稅額撰寫一份 API。稅率級距表如下：
+ 年收入 0 ~ 540,000 : 5%
+ 年收入 540,001 ~ 1,210,000 : 12%
+ 年收入 1,210,001 ~ 2,420,000 : 20%
+ 年收入 2,420,001 ~ 4,530,000 : 30%
+ 年收入 4,530,001 ~ 10,310,000 : 40%
+ 年收入 10,310,001~ : 50%

參考還沒重構前的原始碼，你會怎麼重構？

<br/>

### 原本程式碼
TaxHelper 類別，一長串的 if，不易閱讀
```csharp
public class TaxHelper
{
    public static decimal GetTaxResult(decimal income)
    {
        decimal result = 0;

        if (income > 0 && income <= 540000)
        {
            result += (income - 0) * 0.05m;
        }
        if (income > 540000)
        {
            result += (540000 - 0) * 0.05m;
        }

        if (income > 540000 && income <= 1210000)
        {
            result += (income - 540000) * 0.12m;
        }
        if (income > 1210000)
        {
            result += (1210000 - 540000) * 0.12m;
        }

        if (income > 1210000 && income <= 2420000)
        {
            result += (income - 1210000) * 0.2m;
        }
        if (income > 2420000)
        {
            result += (2420000 - 1210000) * 0.2m;
        }

        if (income > 2420000 && income <= 4530000)
        {
            result += (income - 2420000) * 0.3m;
        }
        if (income > 4530000)
        {
            result += (4530000 - 2420000) * 0.3m;
        }

        if (income > 4530000 && income <= 10310000)
        {
            result += (income - 4530000) * 0.4m;
        }
        if (income > 10310000)
        {
            result += (10310000 - 4530000) * 0.4m;
        }

        if (income > 10310000)
        {
            result += (income - 10310000) * 0.5m;
        }

        return result;
    }
}
```

<br/>

### 用 Chain of Responsibility 重構
其實也不是很漂亮的做法
<br/>TaxHandler 抽象類別
```csharp
public abstract class TaxHandler
{
    protected TaxHandler _nextHandler;

    public void SetNextHandler(TaxHandler handler)
    {
        _nextHandler = handler;
    }

    public abstract decimal CalcTax(decimal income, decimal result);

    protected decimal GetNextResult(decimal income, decimal result)
    {
        if (_nextHandler != null)
        {
            return _nextHandler.CalcTax(income, result);
        }
        else
        {
            return result;
        }
    }
}
```

<br/>級距1 類別實作
```csharp
public class Level1TaxHandler : TaxHandler
{
    public override decimal CalcTax(decimal income, decimal result)
    {
        if (income > 0 && income <= 540000)
        {
            result += (income - 0) * 0.05m;
            return result;
        }
        else
        {
            if (income > 540000)
            {
                result += (540000 - 0) * 0.05m;
            }
            return GetNextResult(income, result);
        }
    }
}
```

<br/>級距2 類別實作
```csharp
public class Level2TaxHandler : TaxHandler
{
    public override decimal CalcTax(decimal income, decimal result)
    {
        if (income > 540000 && income <= 1210000)
        {
            result += (income - 540000) * 0.12m;
            return result;
        }
        else
        {
            if (income > 1210000)
            {
                result += (1210000 - 540000) * 0.12m;
            }
            return GetNextResult(income, result);
        }
    }
}
```

<br/>級距3 類別實作
```csharp
public class Level3TaxHandler : TaxHandler
{
    public override decimal CalcTax(decimal income, decimal result)
    {
        if (income > 1210000 && income <= 2420000)
        {
            result += (income - 1210000) * 0.2m;
            return result;
        }
        else
        {
            if (income > 2420000)
            {
                result += (2420000 - 1210000) * 0.2m;
            }
            return GetNextResult(income, result);
        }
    }
}
```

<br/>級距4 類別實作
```csharp
public class Level4TaxHandler : TaxHandler
{
    public override decimal CalcTax(decimal income, decimal result)
    {
        if (income > 2420000 && income <= 4530000)
        {
            result += (income - 2420000) * 0.3m;
            return result;
        }
        else
        {
            if (income > 4530000)
            {
                result += (4530000 - 2420000) * 0.3m;
            }
            return GetNextResult(income, result);
        }
    }
}
```

<br/>級距5 類別實作
```csharp
public class Level5TaxHandler : TaxHandler
{
    public override decimal CalcTax(decimal income, decimal result)
    {
        if (income > 4530000 && income <= 10310000)
        {
            result += (income - 4530000) * 0.4m;
            return result;
        }
        else
        {
            if (income > 10310000)
            {
                result += (10310000 - 4530000) * 0.4m + (income - 10310000) * 0.5m;
            }
            return GetNextResult(income, result);
        }
    }
}
```

<br/>TaxHelper 類別
```csharp
public class TaxHelper
{
    public static decimal GetTaxResult(decimal income)
    {
        var handler = CreateTaxHabdlersChain();
        decimal result = handler.CalcTax(income, 0);
        return result;
    }

    public static TaxHandler CreateTaxHabdlersChain()
    {
        TaxHandler handler1 = new Level1TaxHandler();
        TaxHandler handler2 = new Level2TaxHandler();
        TaxHandler handler3 = new Level3TaxHandler();
        TaxHandler handler4 = new Level4TaxHandler();
        TaxHandler handler5 = new Level5TaxHandler();
        handler1.SetNextHandler(handler2);
        handler2.SetNextHandler(handler3);
        handler3.SetNextHandler(handler4);
        handler4.SetNextHandler(handler5);
        return handler1;
    }
}
```

<br/>

### 用查表法重構
TaxLevel 類別
```csharp
public class TaxLevel
{
    public decimal LowerBound { get; set; }
    public decimal UpperBound { get; set; }
    public decimal Percentage { get; set; }

    public TaxLevel(decimal lowerBound, decimal upperBound, decimal percentage)
    {
        LowerBound = lowerBound;
        UpperBound = upperBound;
        Percentage = percentage;
    }
}
```

TaxHelper 類別
```csharp
public class TaxHelper
{
    private static List<TaxLevel> _taxRanges = new List<TaxLevel>()
    {
        new TaxLevel(lowerBound: 0, upperBound: 540000, percentage: 0.05m),
        new TaxLevel(lowerBound: 540000, upperBound: 1210000, percentage: 0.12m),
        new TaxLevel(lowerBound: 1210000, upperBound: 2420000, percentage: 0.2m),
        new TaxLevel(lowerBound: 2420000, upperBound: 4530000, percentage: 0.3m),
        new TaxLevel(lowerBound: 4530000, upperBound: 10310000, percentage: 0.4m),
        new TaxLevel(lowerBound: 10310000, upperBound: Int32.MaxValue, percentage: 0.5m)
    };

    public static decimal GetTaxResult(decimal income)
    {
        var result = 0m;
        for (var i = 0; i < _taxRanges.Count - 1; i++)
        {
            var currentTaxLevel = _taxRanges[i];
            if (income > currentTaxLevel.LowerBound && income <= currentTaxLevel.UpperBound)
            {
                result += (income - currentTaxLevel.LowerBound) * currentTaxLevel.Percentage;
            }
            else if (income > currentTaxLevel.UpperBound)
            {
                result += (currentTaxLevel.UpperBound - currentTaxLevel.LowerBound) * currentTaxLevel.Percentage;
            }
            else
            {
                break;
            }
        }

        var lastTaxLevel = _taxRanges[_taxRanges.Count - 1];
        if (income > lastTaxLevel.LowerBound)
        {
            result += (income - lastTaxLevel.LowerBound) * lastTaxLevel.Percentage;
        }

        return result;
    }
}
```

<br/>Chain of Responsibility 較適合用在每個物件差異大的情況，此例雖改用  Chain of Responsibility，但重複程式太多。因為演算都一樣，應把上下界改為參數。
<br/>**變動在哪裡，才是需要設計的地方，所以用了 Chain of Responsibility 是無謂的作法**

---
## 反設計模式範例 - 讀卡機
+ 你要為門禁廠商撰寫 SDK，這家廠商有許多不同型號的讀卡機。
+ 麻煩的是每個讀卡機有相同與不相同的命令協定，基本上沒辦法一個類別搞定。
+ 這 SDK 會散布出去給不特定的客戶使用，我們的目標是要降低開發者的進入障礙。
+ 以下解法將會違反開閉原則與介面隔離原則。
+ 運氣不錯的一點是所有操作回傳值都是 byte[]。

協定表
| R600         | R700         | R800         |
|--------------|--------------|--------------|
| 讀取訊息     | 讀取訊息     | 讀取訊息     |
| 設定時間     | 設定時間     | 設定時間     |
| 寫入單張卡片 | 寫入單張卡片 | 寫入單張卡片 |
|              | 設定營幕顯示 | 設定營幕顯示 |
|              |              | 遠端開門     |

<br/>協定假設為
1. 1 byte 起始碼  0x95)
2. 1 byte 長度碼 (整個 byte[] 的長度)
3. 1 byte card reader id (1~254)
4. 1 byte 命令 (讀取訊息 0x01, 設定時間 0x02, 寫入單張卡片 0x03, 設定螢幕顯示 0x04,遠端開門 0x05)
5. n bytes 資料內容

<br/>

### 原本寫法

模擬從資料庫得到目前的 Readers
```csharp
public class FakeControlsData
{
    public static List<ProtocolBase> GetReaders()
    {
        var list = new List<ProtocolBase>()
        {
            new ProtocolR600() {ReaderId = 1 },
            new ProtocolR600() {ReaderId = 2 },
            new ProtocolR700() {ReaderId = 3 },
            new ProtocolR600() {ReaderId = 4 },
            new ProtocolR600() {ReaderId = 5 },
            new ProtocolR700() {ReaderId = 6 },
            new ProtocolR700() {ReaderId = 7 },
            new ProtocolR800() {ReaderId = 8 }
        };

        return list;
    }
}
```

<br/>以下是 SDK 程式碼
<br/>ProtocolBase 類別
```csharp
public abstract class ProtocolBase
{
    public byte ReaderId { get; set; }
    
    //最大卡片數量
    public abstract ushort MaxCards { get; }
    
    public virtual byte[] ReadMessage()
    {
        byte command = 0x01;
        return GetAllBytes(command, null);
    }

    public virtual byte[] SetDateTime(DateTime time)
    {
        // DateTime轉為 6 byte yy,MM,dd,HH,mm,ss
        byte command = 0x02;
        List<byte> data = new List<byte>();
        data.Add((byte)(time.Year - 2000));
        data.Add((byte)time.Month);
        data.Add((byte)time.Day);
        data.Add((byte)time.Hour);
        data.Add((byte)time.Minute);
        data.Add((byte)time.Second);

        return GetAllBytes(command, data.ToArray());
    }

    public virtual byte[] WriteCard(ushort cardIndex, byte[] cardUID)
    {
        // 2byte card index , 起始值為 0
        // n byte cardUID
        if (cardIndex >= MaxCards)
        {
            return null;
        }
        byte command = 0x03;
        List<byte> data = new List<byte>();
        data.AddRange(BitConverter.GetBytes(cardIndex));
        data.AddRange(cardUID);

        return GetAllBytes(command, data.ToArray());
    }

    protected byte[] GetAllBytes(byte command, byte[] data)
    {
        List<byte> result = new List<byte>();
        result.Add(0x95);
        if (data != null)
        {
            result.Add((byte)(data.Length + 4));
        }
        result.Add(ReaderId);
        result.Add(command);
        if (data != null)
        {
            result.AddRange(data);
        }

        return result.ToArray();
    }
}
```

<br/>ProtocolR600 類別
```csharp
public class ProtocolR600 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 1000;
        }
    }
}
```

<br/>ProtocolR700 類別
```csharp
public class ProtocolR700 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 5000;
        }
    }

    public byte[] SetDisplayText(string text)
    {
        byte command = 0x04;
        byte[] data = Encoding.ASCII.GetBytes(text);
        return GetAllBytes(command, data);
    }
}
```

<br/>ProtocolR800 類別
```csharp
public class ProtocolR800 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 65000;
        }
    }

    public byte[] SetDisplayText(string text)
    {
        byte command = 0x04;
        byte[] data = Encoding.ASCII.GetBytes(text);
        return GetAllBytes(command, data);
    }

    public byte[] OpenDoor()
    {
        byte command = 0x05;
        return GetAllBytes(command, null);
    }
}
```

<br/>以下是 Client 端使用你開發的 SDK 撰寫的程式碼，，可以看到 Client 端須判斷協定是哪種實作才能決定採取什麼動作，假如未來又增加新的讀卡機類別，那麼所有使用 SDK 的 Client 端都要更改，不是很好的做法。
```csharp
var readers = FakeControlsData.GetReaders();
// 如果要求是要 DisplayText
string text = "ABC";

foreach (var reader in readers )
{
    if (reader is ProtocolR600)
    {
        Console.WriteLine($"{reader .GetType().ToString () } 不送資料");
    }

    if(reader is ProtocolR700 )
    {
        var buffer = ((ProtocolR700)reader).SetDisplayText(text);
        
        Console.WriteLine($"{reader.GetType().ToString()} - {reader.ReaderId } 送出 { BitConverter.ToString (buffer)}");
    }

    if (reader is ProtocolR800)
    {
        var buffer = ((ProtocolR800)reader).SetDisplayText(text);
        Console.WriteLine($"{reader.GetType().ToString()} - {reader.ReaderId } 送出 { BitConverter.ToString(buffer)}");
    }
}
```

<br/>其實用反射可以解決問題，但很難期待所有 Client 端都有這麼做
```csharp
var readers = FakeControlsData.GetReaders();

// 如果要求是要 DisplayText
string text = "ABC";
foreach (var reader in readers)
{    
    var method = reader.GetType().GetMethod("SetDisplayText", new Type[] { typeof(string) });
    if (method != null)
    {
        var buffer = (byte[])method.Invoke(reader, new object[] { text });
        Console.WriteLine($"{reader.GetType().ToString()} - {reader.ReaderId } 送出 { BitConverter.ToString(buffer)}");
    }
    else
    {
        Console.WriteLine($"{reader.GetType().ToString() } 不送資料");
    }
}
```

<br/>講師的目的就是撰寫出 SDK，讓 SDK 未來有機會被更改時，不用讓所有 Clinet 端都要更改，不然他會接到廠商的抱怨。

### 採用 Vistor 模式

<br/>修改 SDK
<br/>修改 ProtocolBase 類別
```csharp
public abstract class ProtocolBase
{
    public byte ReaderId { get; set; }
    public abstract ushort MaxCards { get; }
    
    // 使用 Visitor Pattern
    public abstract byte[] GetBytes(IVisitor visitor);  
    
    // 為了 Visitor Pattern, 把 DateTime 綁在這邊
    public DateTime ReaderTime { get; set; }

    // 為了 Visitor Pattern, 把 卡片資料 綁在這邊
    public Tuple<ushort,byte[]> CardInfo { get; set; }

    // 為了 Visitor Pattern, 把 Displaty Text 綁在這邊
    public string DisplayText { get; set; }
}
```

<br/>修改 ProtocolR600 類別
```csharp
public class ProtocolR600 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 1000;
        }
    }

    //這個就是 Visitor pattern 的 Accept
    public override byte[] GetBytes(IVisitor visitor)
    {
       return  visitor.Visit(this);
    }
}
```

<br/>修改 ProtocolR700 類別
```csharp
public class ProtocolR700 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 5000;
        }
    }

    //這個就是 Visitor pattern 的 Accept
    public override byte[] GetBytes(IVisitor visitor)
    {
        return visitor.Visit(this);
    }
}
```

<br/>修改 ProtocolR800 類別
```csharp
public class ProtocolR800 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 60000;
        }
    }

    //這個就是 Visitor pattern 的 Accept
    public override byte[] GetBytes(IVisitor visitor)
    {
        return visitor.Visit(this);
    }
}
```

<br/>IVisitor 介面
```csharp
public interface IVisitor
{
    byte[] Visit(ProtocolR600 reader);
    byte[] Visit(ProtocolR700 reader);
    byte[] Visit(ProtocolR800 reader);
}
```

<br/>ReadMessageVisitor 類別
```csharp
public class ReadMessageVisitor : IVisitor
{
    private byte command = 0x01;

    public byte[] Visit(ProtocolR800 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR700 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR600 reader)
    {
        return GetResult(reader);
    }

    private byte[] GetResult(ProtocolBase reader)
    {
        return ProtocolHelper.GetAllBytes(reader.ReaderId, command, null);
    }
}
```

<br/>SetDateTimeVisitor 類別，寫這一個必須要把 DateTime 綁在 ProtocolBase 身上
```csharp
public class SetDateTimeVisitor : IVisitor
{
    private byte command = 0x02;

    public byte[] Visit(ProtocolR800 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR700 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR600 reader)
    {
        return GetResult(reader);
    }

    private byte[] GetResult(ProtocolBase reader)
    {
        var time = reader.ReaderTime;
        List<byte> data = new List<byte>();
        data.Add((byte)(time.Year - 2000));
        data.Add((byte)time.Month);
        data.Add((byte)time.Day);
        data.Add((byte)time.Hour);
        data.Add((byte)time.Minute);
        data.Add((byte)time.Second);
        return ProtocolHelper.GetAllBytes(reader.ReaderId, command, data.ToArray ());
    }
}
```

<br/>WriteCardVisitor 類別，寫這一個必須要把 cardIndex 和 cardUid 綁在 ProtocolBase 身上
```csharp
public class WriteCardVisitor : IVisitor
{
    private byte command = 0x03;

    public byte[] Visit(ProtocolR800 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR700 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR600 reader)
    {
        return GetResult(reader);
    }

    private byte[] GetResult(ProtocolBase reader)
    {
        if (reader.CardInfo.Item1  >= reader.MaxCards)
        {
            return null;
        }
        List<byte> data = new List<byte>();
        data.AddRange(BitConverter.GetBytes(reader.CardInfo.Item1));
        data.AddRange(reader.CardInfo.Item2 );
       
        return ProtocolHelper.GetAllBytes(reader.ReaderId, command, data.ToArray ());
    }
}
```

<br/>DisplayTextVisitor 類別，寫這一個必須要把 displayText 綁在 ProtocolBase 身上
```csharp
public class DisplayTextVisitor : IVisitor
{
    private byte command = 0x04;

    public byte[] Visit(ProtocolR800 reader)
    {
        return GetResult(reader);
    }

    public byte[] Visit(ProtocolR700 reader)
    {
        return GetResult(reader);
    }

    // R600 不支援 Display Text. 所以回傳 null
    public byte[] Visit(ProtocolR600 reader)
    {
        return null;
    }

    private byte[] GetResult(ProtocolBase reader)
    {
       
        byte[] data = Encoding.ASCII.GetBytes(reader.DisplayText );
        return ProtocolHelper.GetAllBytes(reader.ReaderId,command, data);
    }
}
```

<br/>OpenDoorVisitor 類別
```csharp
public class OpenDoorVisitor : IVisitor
{
    private byte command = 0x05;

    public byte[] Visit(ProtocolR800 reader)
    {
        return GetResult(reader);
    }

    // R700 不支援 OpenDoor. 所以回傳 null
    public byte[] Visit(ProtocolR700 reader)
    {
        return null;
    }

    // R600 不支援 OpenDoor. 所以回傳 null
    public byte[] Visit(ProtocolR600 reader)
    {
        return null;
    }

    private byte[] GetResult(ProtocolBase reader)
    {
        return ProtocolHelper.GetAllBytes(reader.ReaderId, command, null);
    }
}
```

<br/>Client 端程式
```csharp
var readers = FakeControlsData.GetReaders();
// 如果要求是要 DisplayText
string text = "ABC";
var visitor = new DisplayTextVisitor();

foreach (var reader in readers)
{
    reader.DisplayText = text;
    var buffer = reader.GetBytes(visitor);
    if (buffer != null )
    {
        Console.WriteLine($"{reader.GetType().ToString()} - {reader.ReaderId } 送出 { BitConverter.ToString(buffer)}");
    }
    else
    {
        Console.WriteLine($"{reader.GetType().ToString() } 不送資料");
    }
}
```

<br/>

### 反設計模式

<br/>ProtocolBase 類別
```csharp
public abstract class ProtocolBase
{
    public byte ReaderId { get; set; }

    //最大卡片數量
    public abstract ushort MaxCards { get; }

    public virtual byte[] ReadMessage()
    {
        byte command = 0x01;
        return GetAllBytes(command, null);
    }

    public virtual byte[] SetDateTime(DateTime time)
    {
        // DateTime轉為 6 byte yy,MM,dd,HH,mm,ss
        byte command = 0x02;
        List<byte> data = new List<byte>();
        data.Add((byte)(time.Year - 2000));
        data.Add((byte)time.Month);
        data.Add((byte)time.Day);
        data.Add((byte)time.Hour);
        data.Add((byte)time.Minute);
        data.Add((byte)time.Second);
        return GetAllBytes(command, data.ToArray());
    }

    public virtual byte[] WriteCard(ushort cardIndex, byte[] cardUID)
    {
        // 2byte card index , 起始值為 0
        // n byte cardUID
        if (cardIndex >= MaxCards)
        {
            return null;
        }
        byte command = 0x03;
        List<byte> data = new List<byte>();
        data.AddRange(BitConverter.GetBytes(cardIndex));
        data.AddRange(cardUID);
        return GetAllBytes(command, data.ToArray());
    }

    // 在父類別增加接口
    public virtual byte[] SetDisplayText(string text)
    {
        return null;
    }

    // 在父類別增加接口
    public virtual byte[] OpenDoor()
    {
        return null;
    }

    protected byte[] GetAllBytes(byte command, byte[] data)
    {
        List<byte> result = new List<byte>();
        result.Add(0x95);
        if (data != null)
        {
            result.Add((byte)(data.Length + 4));
        }
        result.Add(ReaderId);
        result.Add(command);
        if (data != null)
        {
            result.AddRange(data);
        }
        return result.ToArray();
    }
}
```

<br/>ProtocolR600 類別
```csharp
public class ProtocolR600 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 1000;
        }
    }
}
```

<br/>ProtocolR700 類別
```csharp
public class ProtocolR700 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 5000;
        }
    }

    public byte[] SetDisplayText(string text)
    {
        byte command = 0x04;
        byte[] data = Encoding.ASCII.GetBytes(text);
        return GetAllBytes(command, data);
    }
}
```

<br/>ProtocolR800 類別
```csharp
public class ProtocolR800 : ProtocolBase
{
    public override ushort MaxCards
    {
        get
        {
            return 65000;
        }
    }

    public byte[] SetDisplayText(string text)
    {
        byte command = 0x04;
        byte[] data = Encoding.ASCII.GetBytes(text);
        return GetAllBytes(command, data);
    }

    public byte[] OpenDoor()
    {
        byte command = 0x05;
        return GetAllBytes(command, null);
    }
}
```

<br/>Client 端程式
```csharp
var readers = FakeControlsData.GetReaders();

// 如果要求是要 DisplayText
string text = "ABC";

foreach (var reader in readers)
{
    var buffer = reader.SetDisplayText(text);
    if (buffer != null)
    {
        Console.WriteLine($"{reader.GetType().ToString()} - {reader.ReaderId } 送出 { BitConverter.ToString(buffer)}");
    }
    else
    {
        Console.WriteLine($"{reader.GetType().ToString() } 不送資料");
    }
}
```

<br/>可以使用 Vistor 模式和使用反設計模式的方法都達到同樣的目的，讓 Client 端程式不會因為增加讀卡機而變動，，而兩者比較起來，採用 Vistor 模式要多寫很多程式碼，而反設計模式只在父類別多開兩個接口，所以講師衡量後，是使用反設計模式的做法(OCP 原則，對修改封閉，對擴展開放)就交差了。

<br/>想到要採用某種設計模式，也要想想是否有什麼代價?? 被使用者一直煩??
<br/>違反一點設計原則，但不讓客戶一直煩你