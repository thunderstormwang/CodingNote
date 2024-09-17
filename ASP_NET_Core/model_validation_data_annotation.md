# [.Net Core]模型驗證 - Data Annotations

- [\[.Net Core\]模型驗證 - Data Annotations](#net-core模型驗證---data-annotations)
  - [類別層級驗證](#類別層級驗證)
  - [Value Type 掛 Required，沒起到必填的效果](#value-type-掛-required沒起到必填的效果)
  - [Non Nullable Property 掛 \[Required\] 在 .Net Core 6.0 的預設行為，容易踩雷](#non-nullable-property-掛-required-在-net-core-60-的預設行為容易踩雷)
  - [撰寫模型驗證的單元測試](#撰寫模型驗證的單元測試)


使用平台: .Net Core 6.0

---

## 類別層級驗證

參數 Model，  
且有個驗證需求是 Email 和 Phone 至少要輸入一個  

```csharp
public class Person
{
    [Required]
    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }
}
```

那麼我們讓此 model 繼承 IValidatableObject, 並選寫自訂驗證

```csharp
public class Person : IValidatableObject
{
    [Required]
    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (string.IsNullOrEmpty(Email) && string.IsNullOrEmpty(Phone))
        {
            yield return new ValidationResult( $"At least one of Email, Phone is required.", 
                new List<string>() { $"{nameof(Email)}", $"{nameof(Phone)}" });
        }
    }
}
```

如此就實現 Email 和 Phone 至少要輸入一個的驗證  
順序上屬性的驗證要先全通過, 類別層級的驗證才會被呼叫到  
也就是說如果 Name, Email, Phone 都沒填，只會先看到  
>The Name field is required.

將 Name 填入後, 才會看到
>At least one of Email, Phone is required.

---

## Value Type 掛 Required，沒起到必填的效果

宣告這樣的參數 model  

```csharp
public class Person
{
    [Required]
    public int Age { get; set; }

    public string Name { get; set; }
}
```

我們故意不傳入 Age，會發現仍可通過 model validation
```json
{
    "name": "John"
}
```

model binding 時如果 value type 的 property 對應不到，會給予它預設值，例如 int 會給 0，此時 `[Required]` 無法區分該 property 是沒給值還是傳入的值與預設值相同，所以必填就不起作用。

改成 `int?` 即可讓必填就起作用。

---

## Non Nullable Property 掛 [Required] 在 .Net Core 6.0 的預設行為，容易踩雷

在 [[C# 8.0]Nullable Refence Type](../csharp/nullable_reference_type.md) 提到 reference type 也開始引入 nullable。


這樣的參數 model  
```csharp
public class Person
{
    public string Name { get; set; }

    public int Age { get; set; }
}
```
在以前如果只傳入 Age，我們會預期會通過 model validation。  

```json
{
    "age": 18
}
```

但是在 .Net Core 6.0 後會得到錯誤訊息
> The Name field is required.

這是因為 .Net Core 6.0 的 csproj 檔案預設是有這行的 `<Nullable>enable</Nullable>`，預設 reference type 也要有值

依照官網說明，沒掛  ``` [Required] ``` 的 reference type 會預設當作有掛 ``` [Required(AllowEmptyStrings = true)] ```

所以我們若傳入，就可以通過 model validation
```json
{
    "name": "",
    "age": 18
}
```

如果你的本意是連空字串都不行，那就加上 `[Required]`，那行為就跟 6.0 以前一樣。  

如果是希望 Name 跟以前一樣，不為必填，那可以這樣做  

方法一，csproj 的 `<Nullable>enable</Nullable>` 拿掉，讓 reference type 可以連名字都不給就傳進來，但這最好在開發初期做  

方法二，在 program.cs 關閉預設掛 `[Required(AllowEmptyStrings = true)]` 的行為，讓 model validation 無視該參數沒被傳入或是被設為 null 的情況  

```csharp
builder.Services.AddControllers(options =>
    {
        options.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes = true;
    });
```

方法三，或是將該 property 從 `string` 改為 `string?`

---

## 撰寫模型驗證的單元測試

使用平台: .Net Core 7.0

使用套件:
- FluentAssertions
- NUnit

一個這樣的參數 model 驗證

```csharp
public class Person
{
    [Required(ErrorMessage = "姓名未填寫")]
    public string Name { get; set; }

    public int Age { get; set; }
}
```

單元測試可以這樣寫

```csharp
public void TestMethod()
{
    var request = new Person()
    {
        Name = null,
        Age = 20
    };
    
    var actual = new List<ValidationResult>();
    var ctx = new ValidationContext(request);
    Validator.TryValidateObject(request, ctx, actual, true);
    actual.Should().NotBeNullOrEmpty();
    actual.Count().Should().Be(1);
    actual.First().MemberNames.Should().BeEquivalentTo(new List<string>() {$"{nameof(request.Name)}" });
    actual.First().ErrorMessage.Should().Contain("姓名未填寫");
}
```

---

參考自
- [Model validation in ASP.NET Core MVC and Razor Pages](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-6.0)