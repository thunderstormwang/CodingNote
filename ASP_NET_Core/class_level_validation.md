# [驗證]自訂類別層級驗證

使用平台: .Net Core 6.0

---

繼承 ValadationAttribute，撰寫自訂驗證規則：至少輸入一個欄位，達成後端驗證

<br/>以此 Model，Email 和 Phone 至少要輸入一個
```csharp
public class Person
{
    [Required]
    public string Name { get; set; }

    public string Email { get; set; }

    public string Phone { get; set; }
}
```

<br/>那麼我們讓此 model 繼承 IValidatableObject, 並選寫自訂驗證
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

<br/>如此就實現 Email 和 Phone 至少要輸入一個的驗證
<br/>順序上屬性的驗證要先全通過, 類別層級的驗證才會被呼叫到
<br/>也就是說如果 Name, Email, Phone 都沒填，只會先看到
>The Name field is required.

<br/>將 Name 填入後, 才會看到
>At least one of Email, Phone is required.