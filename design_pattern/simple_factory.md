# Design Pattern - Simple Factory

+ 利用分支運算( if else, switch case)決定實體
+ 如果要增加類別，就必須修改程式碼( if else, switch case)，這個動作有些人認為違反開閉原則
+ 改善分支運算的問題
  + 使用資源字典
  + 使用 Attribute

## Simple Factory for Adapter

### 利用分支運算決定實體

<br/>工廠類別
```csharp
public enum CommucationType
{
    Tcp,
    SerialPort
}

public class CommucationFactory
{
    public static ICommunication GetInstance(CommucationType type)
    {
        switch (type)
        {
            case CommucationType.Tcp:
                return new TcpCommunication();
            case CommucationType.SerialPort:
                return new SerialCommunication();
            default:
                throw new ArgumentOutOfRangeException();
        }
    }
}
```

<br/>產品介面，也 Adapter 的 ITarget
```csharp
public interface ICommunication
{
    bool Connect(string target);
    void Disconnect();
    void SendData(byte[] buffer);
    byte[] ReceiveData();
}
```

<br/>產品介面，Tcp Adapter，虛擬碼
```csharp
public class TcpCommunication : ICommunication
{
    public bool Connect(string target)
    {
        Console.WriteLine(string.Format("Tcp 連接到 : {0} ", target));
        return true;
    }

    public void Disconnect()
    {
        Console.WriteLine("Tcp 連接關閉");
    }

    public byte[] ReceiveData()
    {
        byte[] result = new byte[] { 0x11, 0x12, 0x13, 0x14 };
        return result;
    }

    public void SendData(byte[] buffer)
    {
        string data = BitConverter.ToString(buffer);
        Console.WriteLine(string.Format("Tcp 送出 : {0} ", data));
    }
}
```

<br/>產品介面，Serialport Adapter, 虛擬碼
```csharp
public class SerialCommunication : ICommunication
{
    public bool Connect(string target)
    {
        Console.WriteLine(string.Format("開啟 Serial Port : {0} ", target));
        return true;
    }

    public void Disconnect()
    {
        Console.WriteLine("關閉 Serial Port");
    }

    public byte[] ReceiveData()
    {
        byte[] result = new byte[] { 0x01, 0x02, 0x03, 0x04 };
        return result;
    }

    public void SendData(byte[] buffer)
    {
        string data = BitConverter.ToString(buffer);
        Console.WriteLine(string.Format("Serial Port 送出 : {0} ", data));
    }
}
```

<br/>Client 端程式
```csharp
var commucation = CommucationFactory.GetInstance(CommucationType.Tcp);
commucation.Connect("192.168.0.1:8888");
```

### 利用 Attribute 決定實體
<br/>工廠類別
```csharp
public class CommunicationAttribute : Attribute
{
    public CommunicationAttribute(Type type)
    {
        InstanceType = type;
    }

    public Type InstanceType
    {
        get; private set;
    }
}

public enum CommunicationType
{
    [Communication(typeof(TcpCommunication))]
    Tcp,
    [Communication(typeof(SerialCommunication))]
    SerialPort
}
```

<br/>Helper，取得 enum CommunicationType 上的 Attribute 中的 InstanceType 屬性值
```csharp
public class CommunicationTypeHelper
{
    public static Type GetInstanceType(CommunicationType type)
    {
        FieldInfo data = typeof(CommunicationType).GetField(type.ToString());
        Attribute attribute = Attribute.GetCustomAttribute(data, typeof(CommunicationAttribute));
        CommunicationAttribute result = (CommunicationAttribute)attribute;
        return result.InstanceType;
    }
}
```

<br/>工廠類別，運用 Attribute 決定該建立的實體
```csharp
public class CommunicationFactory
{
    public static ICommunication GetInstance(CommunicationType type)
    {
        Type t = CommunicationTypeHelper.GetInstanceType(type);
        return (ICommunication)Activator.CreateInstance(t);
    }
}
```

<br/>產品類別、Client 端程式 同上

### 利用資源字典決定實體

<br/>建立資源搜尋器
```csharp
public enum CommunicationType
{
    Tcp,
    SerialPort
}

public class CommunicationResources
{
    private static Dictionary<CommunicationType, Type> _resources;

    static CommunicationResources()
    {
        _resources = new Dictionary<CommunicationType, Type>();
        _resources[CommunicationType.Tcp] = typeof(TcpCommunication);
        _resources[CommunicationType.SerialPort] = typeof(SerialCommunication);
    }

    public static Type GetInstanceType(CommunicationType type)
    {
        if (_resources.ContainsKey(type))
        {
            return _resources[type];
        }
        else
        {
            throw new ArgumentNullException();
        }
    }
}
```

<br/>工廠類別，運用資源搜尋法
```csharp
public class CommunicationFactory
{
    public static ICommunication GetInstance(CommunicationType type)
    {
        var t = CommunicationResources.GetInstanceType(type);
        return (ICommunication)Activator.CreateInstance(t);
    }
}
```

<br/>產品類別、Client 端程式 同上
