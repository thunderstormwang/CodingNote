# 撰寫驗證的單元測試

使用平台: .Net Core 7.0

使用套件:
- FluentAssertions
- NUnit

一個這樣的 model 驗證

```csharp
public class Person
{
    [Required(ErrorMessage = "姓名未填寫")]
    public string Name { get; set; }

    public int Age { get; set; }
}
```

可以這樣寫單元測試

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
        actual.First().MemberNames.Should().BeEquivalentTo(new List<string>() { $"{nameof(request.Name)}" });
        actual.First().ErrorMessage.Should().Contain("姓名未填寫");
    }
```

