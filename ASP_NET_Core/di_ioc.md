# [.Net Core]Dependency Injection、Inversion of Control

## IOC 為一種概念  
- A 物件程式內部需要使用 B 物件，A 依賴 B  
- 控制反轉把原本 A 對 B 直接控制權移交給由第三方容器，或者說，由呼叫 A 的外部程式來傳入 B 給 A  
- 降低 A 對 B 物件的耦合程度，並讓雙方都倚賴抽象。  
- DI 為一種實現方式  

## 何謂 Dependency Injection

當 UserService 在內部會使用到 MysqlDatabaseClient、SmsMessenger 時，我們可能在初始化 UserService 時，直接建立 MysqlDatabaseClient、SmsMessenger 物件，這樣的做法會讓 UserService 與 MysqlDatabaseClient、SmsMessenger 之間產生耦合，當 MysqlDatabaseClient、SmsMessenger 有變動時，UserService 也需要跟著變動。  

```csharp
public class UserService { 
    private MysqlDatabaseClient dbClient = new MysqlDatabaseClient(); 
    private SmsMessenger messenger = new SmsMessenger(); 
    // ... 
    
    public void changePassword(User user, String newPassword) 
    { 
        try 
        { 
            dbClient.updatePassword(user.getId(), newPassword); 
            messenger.send(user, TextTemplate.PASSWORD_CHANGE_NOTICE); 
        }
        catch (OperationException e) 
        {
            throw new AppRuntimeException("Cannot change password", e);
        } 
    } 
}
```

透過 DI，我們可以將 MysqlDatabaseClient、SmsMessenger 透過建構子傳入 UserService，讓 UserService 不需要知道 MysqlDatabaseClient、SmsMessenger 的實作，只需要知道 MysqlDatabaseClient、SmsMessenger 的介面即可。  

```csharp
public class UserService { 
    private DatabaseClient dbClient; 
    private Messenger messenger; 
    
    public UserService(DatabaseClient dbClient, Messenger messenger) 
    {
        dbClient = dbClient; 
        messenger = messenger;
    }

  // ...  changePassword method 不變 
}
```

DI 的常見方式  
- Constructor Injection  
- Setter Injection  
- Field Injection  
- Interface Injection  

DI 的優點  
- 解決相依性，只要 Interface 的介面沒改，或者替換實作該 Interface 的實體，都不會須要更動到使用此 Interface 的程式碼。  
- 更清晰的意圖，查看在建構式被注入什麼，即可知道程式使用到什麼模組、會怎麼運作。
- 可維護性提高  
- 容易測試，容易將使用外部資源的程式碼替掉換，例如：DB、第三方 API  

DI 的缺點  
- trace code 時沒法直接看出實作是什麼  
- 要寫的程式碼變多  

.Net Core 的 DI 框架  
- Autofac  
- Unity  
- Microsoft.Extensions.DependencyInjection  