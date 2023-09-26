# [C# 2.0]將例外丟給上層，且不破壞 stack trace 結構

將例外丟出，可能會破壞 stack trace 結構，做了些測試後，記錄如下

結論：這種寫法可保留 stack trace 結構

```csharp
throw new Exception("Additional Information", ex);
```

---

程式結構如下：
<br/>用兩層 try/catch，故意在內層引發例外，然後內層的 try/catch 將例外拋出( throw )，由外層接住此例外並記錄，看看哪種寫法能真正記到引發例外的行數

```csharp
try
{
    try
    {
        在此處引發例外
    }
    catch
    {
        將例外拋出( throw )
    }
}
catch
{
    在此接住例外並記錄
}
```

<br/>故意製造除以零的例外
```csharp
public int DevideByZero()
{
    int a = 500;
    int b = 0;
    return a / b;
}
```

<br/>用三種不同的寫法將例外丟給上層

### throw ex
無法保留原本出問題的行數
```csharp
try
{
    try
    {
        // stack trace 記不到這行
        DevideByZero();
    }
    catch(Exception ex)
    {
        // stack trace 只記到這行
        throw ex
    }
}
catch(Exception ex)
{
    Console.WriteLine(ex.ToString());
}

public int DevideByZero()
{
    int a = 500;
    int b = 0;
    // stack trace 也沒記到真正除以 0 的行數
    return a / b;
}
```

### throw
部份資訊遺失
```csharp
try
{
    try
    {
        // stack trace 沒記到這行
        DevideByZero();
    }
    catch(Exception ex)
    {
        // stack trace 有記到這行
        throw;
    }
}
catch(Exception ex)
{
    Console.WriteLine(ex.ToString());
}

public int DevideByZero()
{
    int a = 500;
    int b = 0;
    // stack trace 有記到真正除以 0 的行數
    return a / b;
}
```

### throw new Exception
完整保留原本出問題的行數
```csharp
try
{
    try
    {
        // stack trace 有記到這行
        DevideByZero();
    }
    catch(Exception ex)
    {
        // stack trace 有記到這行
        throw new Exception("Additional Information", ex);
    }
}
catch(Exception ex)
{
    Console.WriteLine(ex.ToString());
}

public int DevideByZero()
{
    int a = 500;
    int b = 0;
    // stack trace 有記到真正除以 0 的行數
    return a / b;
}
```