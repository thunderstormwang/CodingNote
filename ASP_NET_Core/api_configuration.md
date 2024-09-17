# [.Net Core]Configuration

使用框架：.Net Core 8.0  

## 注入設定檔

在 appsetting.json 設定如下的設定檔，加入 Member 區塊

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Member": {
    "Id": "TestName",
    "Url": [
      "http://localhost:5000",
      "http://localhost:5001"
    ]
  }
}
```

設定對應的 C# 類別

```csharp
public class Member
{
    public string? Id { get; set; }
    public List<string>? Url { get; set; }
}
```

有以下方法可注入設定檔

### 一、直接使用 IConfiguration

IConfiguration 是內建的 DI 服務，可以直接注入使用  
但是需要自己處理型別轉換

```csharp
public class ServiceClass
{
    private readonly Member _member;

    public ServiceClass(IConfiguration configuration)
    {
        _member = new Member
        {
            Id = configuration.GetValue<string>("Member:Id"),
            Url = configuration.GetSection("Member:Url").Get<List<string>>()
        };
    }
}
```

### 二、強型別設定，使用 IOptions

在初始化時，將設定檔注入 IOptions，以下為注入的方式  
在 program.cs 指定設定檔的位置和注入的型別

```csharp
var builder = WebApplication.CreateBuilder(args);

IConfiguration configuration = builder.Configuration;
builder.Services.Configure<Member>(configuration.GetSection("Member"));

var app = builder.Build();
```

在 ServiceClass 取得該設定檔物件  
注意這裡的型別是 IOptions<Member>，而不是 Member  

```csharp
public class ServiceClass
{
    private readonly Member _member;

    public ServiceClass(IOptions<Member> member)
    {
        _member = member.Value;
    }
}
```

### 三、強型別設定，直接使用類別

在初始化時，將設定檔轉換為你指定的類別，再注入，以下為注入的方式  
在 program.cs 指定設定檔的位置和注入的型別

```csharp
var builder = WebApplication.CreateBuilder(args);

var memberConfig = configuration.GetSection("Member").Get<Member>();
builder.Services.AddSingleton<Member>(memberConfig);

var app = builder.Build();
```

在 ServiceClass 取得該設定檔物件  
注意這裡的型別已是 Member  

```csharp
public class ServiceClass
{
    private readonly Member _member;

    public ServiceClass(Member member)
    {
        _member = member;
    }
}
```

## 設定 Config 路徑

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Configuration.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true);
builder.Configuration.AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true, reloadOnChange: true);
```

- path：組態檔案的路徑位置。
- optional：如果是必要的組態檔，optional 就要設定為 false，當檔案不存在就會拋出 FileNotFoundException。
- reloadOnChange：如果檔案被更新，就同步更新 IConfiguration 實例的值。