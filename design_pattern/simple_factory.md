# Design Pattern - Simple Factory

+ 利用分支運算( if else, switch case)決定實體
+ 如果要增加類別，就必須修改程式碼( if else, switch case)，這個動作有些人認為違反開閉原則

<br/>工廠類別
```csharp
public enum CommucationType
{
    Tcp,
    SerialPort
}

/// <summary>
/// 使用簡單分支運算
/// </summary>
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

<br/>產品類別
```csharp
/// <summary>
/// 這就是一個 Target
/// </summary>
public interface ICommunication
{
    bool Connect(string target);
    void Disconnect();
    // 被迫改名
    void SendData(byte[] buffer);
    byte[] ReceiveData();
}
```

```csharp
/// <summary>
/// Tcp Adapter, 虛擬碼
/// </summary>
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

```csharp
/// <summary>
/// Serialport Adapter, 虛擬碼
/// </summary>
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