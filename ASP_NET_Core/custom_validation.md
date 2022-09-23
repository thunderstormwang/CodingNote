# [驗證]自訂驗證(包含前後端) 至少輸入一個欄位

4個步驟
1. 繼承 ValadationAttribute，達成後端驗證
2. 繼承 IClientValidatable，將 Html5 data-* 的 attribute 加在前端
3. 寫自訂的 js 驗證函式
4. 寫 adapter 使 js 驗證函式看得懂

<br/>Model
```csharp
public class AddBannerCommand
{
    [Required]
    public DateTime StartTime { get; set; }

    [Required]
    public DateTime EndTime { get; set; }

    [Required]
    public string Title { get; set; }

    [Required]
    public string Picture { get; set; }

    [AtLeastOneRequired(nameof(CampaignId), nameof(PxBoxPath), ErrorMessage = "活動頁 Id, 稱購路徑至少須填入一個")]
    public int? CampaignId { get; set; }
    
    /// <summary>
    /// 箱購路徑
    /// </summary>
    public string PxBoxPath { get; set; }
}
```

<br/>宣告 AtLeastOneRequiredAttritube，並繼承ValadationAttribute、IClientValidatable
```csharp
public class AtLeastOneRequiredAttribute : ValidationAttribute
{
    private readonly string[] _properties;

    /// <inheritdoc />
    public AtLeastOneRequiredAttribute(params string[] properties)
    {
        _properties = properties;
    }

    /// <inheritdoc />
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (_properties == null || _properties.Length < 1)
        {
            return null;
        }

        foreach (var item in _properties)
        {
            var propertyInfo = validationContext.ObjectType.GetProperty(item);
            if (propertyInfo == null)
            {
                return new ValidationResult($"unknown property {item}");
            }

            var propertyValue = propertyInfo.GetValue(validationContext.ObjectInstance, null);
            if (propertyValue is string stringValue)
            {
                if (!string.IsNullOrEmpty(stringValue))
                {
                    return null;
                }
            }
            else if (propertyValue != null)
            {
                return null;
            }
        }

        return new ValidationResult(FormatErrorMessage(validationContext.DisplayName));
    }
}
```