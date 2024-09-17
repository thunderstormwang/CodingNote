# [套件]FluentValidation

使用平台: .Net Core 8.0

## 使用方法

參數 Model:
```csharp
public class CreateCurrencyDto
{
    public string Code { get; set; }

    public string CurrencyName { get; set; }
}
```

為該參數撰寫驗證規則，須繼承 `AbstractValidator<T>`，然後在建構式指定你要的驗證規則，`ClassLevelCascadeMode` 這裡設定為 `CascadeMode.Stop`，表示當有一個驗證失敗時，就不再繼續驗證。
```csharp
public class CreateCurrencyDtoValidator : AbstractValidator<CreateCurrencyDto>
{
    public CreateCurrencyDtoValidator()
    {
        ClassLevelCascadeMode = CascadeMode.Stop;

        RuleFor(request => request.Code).NotNull().WithMessage(r => $"{nameof(r.Code)} 必填")
            .NotEmpty().WithMessage(r => $"{nameof(r.Code)} 必填");

        RuleFor(request => request.CurrencyName).NotNull().WithMessage(r => $"{nameof(r.CurrencyName)} 必填")
            .NotEmpty().WithMessage(r => $"{nameof(r.CurrencyName)} 必填");
    }
}
```

在 program.cs 中註冊 FluentValidation
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers()

builder.Services.AddFluentValidationAutoValidation(opt =>
{
    // 關閉預設的 Data Annotations 棤型驗證
    opt.DisableDataAnnotationsValidation = true;
});

// 一次註冊所有用 FluentValidation 撰寫的驗證
builder.Services.AddValidatorsFromAssemblyContaining<CreateCurrencyDtoValidator>();

var app = builder.Build();

app.Run();
```

至於當模型驗證失敗時，怎麼自訂回傳結構，請參考 [[.Net Core]ActionFilter 應用 - 自訂 Api 模型驗證回傳結構](api_custom_validation_response.md)。

## 如何寫單元測試

在 FluentValidation 有提供 `TestValidate` 方法，可以用來測試驗證是否正確，以下是一個範例

```csharp
    private CreateCurrencyDtoValidator _validator = new CreateCurrencyDtoValidator();

public void Test()
{
    var request = new CreateCurrencyDto
    {
        Code = string.Empty,
        CurrencyName = "新台幣"
    };

    var validator = new CreateCurrencyDtoValidator();
    var result = validator.TestValidate(request);
    result.ShouldHaveValidationErrorFor(x => x.Code).WithErrorMessage($"{nameof(CreateCurrencyDto.Code)} 必填");
}
```

我覺得它的測試方法比較直覺，不需要像 DataAnnotations 那樣要用 `Validator.TryValidateObject` 來測試。  