# Model Validation Required Attribute

使用平台: .Net Core 6.0

---

## Value Type 掛 Required，沒起到必填的效果

宣告這樣的 model，會發現即使不傳 Age 仍可通過 model validation

```csharp
public class Person
{
    [Required]
    public int Age { get; set; }
}
```

model binding 時如果 value type 的 property 對應不到，會給予它預設值，例如 int 會給 0，此時 ``` [Required] ``` 無法區分該 property 是沒給值還是傳入的值與預設值相同，所以必填就不起作用。

改成 ``` int? ``` 即可讓必填就起作用。


## Non Nullable Property 掛 [Required] 在 .Net Core 6.0 的預設行為，容易踩雷

在 [[C# 8.0]Nullable Refence Type](../csharp/nullable_reference_type.md) 提到 reference type 也開始引入 nullable。


這樣的 model，在以前如果只傳入 Age，我們會預期會通過 model validation。
```csharp
public class Person
{
    public string Name { get; set; }

    public int Age { get; set; }
}
```
但是在 .Net Core 6.0 後會得到錯誤訊息
> The Name field is required."

這是因為 .net core 6.0 的 csproj 檔案預設是有這行的 ``` <Nullable>enable</Nullable> ```，預設 reference type 也要有值

依照官網說明，沒掛  ``` [Required] ``` 的 reference type 會預設當作有掛 ``` [Required(AllowEmptyStrings = true)] ```

所以我們若傳入，就可以通過
```json
{
    "name": ""
}
```

<br/>如果你的本意是連空字串都不行，那就加上 ``` [Required] ```，那行為就跟 6.0 以前一樣。

如果是希望 Name 跟以前一樣，不為必填，那可以這樣做

- csproj 的 ``` <Nullable>enable</Nullable> ``` 拿掉，讓 reference type 可以連名字都不給就傳進來，但這最好在開發初期做
- SuppressImplicitRequiredAttributeForNonNullableReferenceTypes 改為 true，關閉預設掛 ``` [Required(AllowEmptyStrings = true)] ``` 的行為
```csharp
builder.Services.AddControllers(options =>
    {
        options.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes = true;
    });
```
- 或是將該 property 從 ``` string ``` 改為 ``` string? ```

---

參考自
- [Model validation in ASP.NET Core MVC and Razor Pages](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-6.0)