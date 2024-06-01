# [套件] AutoMapper

經過測試

丟 null 給它，它也會回你 null 物件  
得到的 dto 也是 null  
```csharp
var dto = mapper.Map<Destination>(null);
```

---

可以寫測試  
```csharp
var config = new MapperConfiguration(cfg => cfg.CreateMap<Source, Destination>());
config.AssertConfigurationIsValid();
```

可檢查出
- 目標物件有哪個 Property 沒被對到
- 型別對不上，例如想將 String 轉成 List<String>，如果想將基本型別互轉，例如想將 String 轉成 Int，是可以通過測試，但在 runtime 可能出錯，例如傳入 "ABC" 是不能轉成 Int 的

---

承上，型別對不上，可提供自己寫的 CustomTypeConverter，  
或是更複雜的轉換，就提供自己寫 CustomValueResolver。  