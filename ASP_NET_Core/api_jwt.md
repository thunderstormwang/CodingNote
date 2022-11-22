# 在 .Net Core WebApi 使用 JWT 驗證

使用平台: .Net Core 6.0

使用套件:
- Microsoft.AspNetCore.Authentication.JwtBearer

---

需求：

有以下角色：Administrator、Teacher、Student

各角色以下權限

 | 功能     | 角色                          |
 |--------|-------------------------------|
 | 增加帳號 | Administrator                 |
 | 解鎖帳號 | Administrator                 |
 | 請假     | Administrator、Teacher、Student |
 | 設定課程 | Administrator、Teacher         |
 | 取消課程 | Administrator、Teacher         |
 | 選修課程 | Administrator、Student         |
 | 退選課程 | Administrator、Student         |

<br/>增加 AuthSetting Model
```csharp
public class AuthSetting
{
    public string Secret { get; set; }

    public int ExpirationInMinutes { get; set; }

    public string Issuer { get; set; }

    public string Audience { get; set; }
}
```

<br/>加入角色 enum
```csharp
public enum MyRole
{
    Administrator = 0,
    Teacher = 1,
    Student = 2
}
```

<br/>增加 JwtHelper, 負責生成 jwt token
```csharp
public class JwtHelper
{
    private readonly AuthSetting _authSetting;

    public JwtHelper(IConfiguration config)
    {
        _authSetting = config.GetSection("AuthSetting").Get<AuthSetting>();
    }

    public string GenerateSecurityToken(int userId, string userDisplayName, string email, List<MyRole> roles)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_authSetting.Secret);

        var claims = new List<Claim>()
        {
            // 通常在系統中是唯一的識別碼
            new Claim(JwtRegisteredClaimNames.Sub, userId.ToString()),
            // 可以塞自訂的資料
            new Claim("display_name", userDisplayName),
            new Claim(JwtRegisteredClaimNames.Iss, _authSetting.Issuer),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new Claim(JwtRegisteredClaimNames.Email, email)
        };
        // 塞入角色
        claims.AddRange(roles.Select(r => new Claim("role", r.ToString())));
        
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims.ToArray()),
            Audience = _authSetting.Audience,
            Expires = DateTime.UtcNow.AddMinutes(_authSetting.ExpirationInMinutes),
            SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);

        return tokenHandler.WriteToken(token);
    }
}
```

<br/>appsetting.json 加上
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "AuthSetting": {
    "Secret": "RFv7DrqznYL6nv7DrqdfrenQYO9JxIsWdcjnQYL6nu09f",
    "ExpirationInMinutes": 1440,
    "Issuer": "myIssuer",
    "Audience": "myAudience"
  }
}

```

<br/>增加 AuthenticationExtension, 負責驗證 jwt token
```csharp
public static class AuthenticationExtension
{
    public static IServiceCollection AddTokenAuthentication(this IServiceCollection services, IConfiguration config)
    {
        var authSetting = config.GetSection("AuthSetting").Get<AuthSetting>();
        var key = Encoding.ASCII.GetBytes(authSetting.Secret);
        
        services.AddAuthentication(x =>
            {
                x.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                x.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
            })
            .AddJwtBearer(x =>
            {
                x.TokenValidationParameters = new TokenValidationParameters
                {
                    IssuerSigningKey = new SymmetricSecurityKey(key),
                    ValidateIssuer = true,
                    ValidIssuer = authSetting.Issuer,
                    ValidateAudience = true,
                    ValidAudience = authSetting.Audience,
                    ValidateLifetime = true
                };
            });

        return services;
    }
}
```

<br/>修改 program.cs 加上指定用 AuthenticationExtension
```csharp
var configuration = builder.Configuration;

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddTokenAuthentication(configuration);
```

<br/>呼叫 app.UseAuthentication()，注意順序很重要，要先驗證才能驗權限
```csharp
app.UseHttpsRedirection();

// 注意順序很重要
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

<br/>修改 program.cs，在 swagger 使用 jwt token 驗證
```csharp
builder.Services.AddSwaggerGen(option =>
{
    option.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme()
    {
        Description = "放 jwt token",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Scheme = "bearer",
        Type = SecuritySchemeType.Http,
        BearerFormat = "jwt"
    });
    option.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new List<string>()
        }
    });
});
```

<br/>增加 LoginRequest Model
```csharp
public class LoginRequest
{
    [Required]
    public string? Account { get; set; }
    
    [Required]
    public string? Password { get; set; }
    
    [Required]
    public string? Name { get; set; }
}
```

<br/>增加登入的 action, 回傳 token
 ```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : Controller
{
    private IConfiguration _config;

    public AuthController(IConfiguration config)
    {
        _config = config;
    }

    [HttpPost, Route("Login")]
    public IActionResult Login(LoginRequest request)
    {
        var accountService = new AccountService();
        var user = accountService.AuthenticateUser(request.Account, request.Password);
        if (user == null)
            throw new ApiException("Invalid Login Credentials: " + bus.ErrorMessage, 401);
        
        var jwtHelper = new JwtHelper(_config);
        var token = jwtHelper.GenerateSecurityToken(user.Id, user.DisplayName, "fake@email.com", user.Roles);
        return Ok(token);
    }
}
```

<br/>打 ``` /api/Auth/Login ``` 得到的 token，解出來會像下面的結構
```json
{
  "sub": "string",
  "userId": "99",
  "iss": "myIssuer",
  "jti": "df339693-0185-4bbc-ad7a-ec201f27a4bb",
  "email": "fake@email.com",
  "role": [
    "Teacher",
    "Student"
  ],
  "nbf": 1667826447,
  "exp": 1667912847,
  "iat": 1667826447,
  "aud": "myAudience"
}
```

<br/>增加 AnonymousController，有以下 Action
```csharp
[AllowAnonymous]
[ApiController]
[Route("api/[controller]")]
public class AnonymousController : Controller
{
    [HttpGet, Route("anonymous")]
    public IActionResult Anonymous()
    {
        return Ok("anonymous");
    }
}
```

<br/>增加 PermissionController，加入以下 Action
```csharp
[ApiController]
[Route("api/[controller]")]
public class PermissionController : Controller
{
    [Authorize]
    [HttpGet, Route("authorize")]
    public IActionResult Authorize()
    {
        return Ok("authorize");
    }
}
```

<br/>增加 AdministrativeController，加入以下 Action
```csharp
[ApiController]
[Route("api/[controller]")]
public class AdministrativeController : Controller
{
    [Authorize(Roles = "Administrator")]
    [HttpPost, Route("add_account")]
    public IActionResult AddAccount([FromBody] Course course)
    {
        return Ok($"{nameof(AddAccount)} OK");
    }
    
    [Authorize(Roles = "Administrator")]
    [HttpPost, Route("unfrozen_account")]
    public IActionResult UnfrozenAccount([FromBody] Course course)
    {
        return Ok($"{nameof(UnfrozenAccount)} OK");
    }
    
    [Authorize(Roles = "Administrator,Teacher,Student")]
    [HttpPost, Route("dayoff")]
    public IActionResult DayOff([FromBody] Course course)
    {
        return Ok($"{nameof(DayOff)} OK");
    }
}
```

<br/>增加 CourseController，加入以下 Action
```csharp
[ApiController]
[Route("api/[controller]")]
public class CourseController : Controller
{
    [Authorize(Roles = "Administrator,Teacher")]
    [HttpPost, Route("set_course")]
    public IActionResult SetCourse([FromBody] Course course)
    {
        return Ok($"{nameof(SetCourse)} OK");
    }
    
    [Authorize(Roles = "Administrator,Teacher")]
    [HttpPost, Route("cancel_course")]
    public IActionResult CancelCourse([FromBody] Course course)
    {
        return Ok($"{nameof(CancelCourse)} OK");
    }
    
    [Authorize(Roles = "Administrator,Student")]
    [HttpPost, Route("major_course")]
    public IActionResult MajorCourse([FromBody] Course course)
    {
        return Ok($"{nameof(MajorCourse)} OK");
    }
    
    [Authorize(Roles = "Administrator,Student")]
    [HttpPost, Route("withdraw_course")]
    public IActionResult WithdrawCourse([FromBody] Course course)
    {
        return Ok($"{nameof(WithdrawCourse)} OK");
    }
}
```

將專案跑 debug mode，把 swagger 開起來

``` /api/anonymous/anonymous ``` 沒掛驗證 filter，不用帶 token 也可以通過，得到回覆
>anonymous

<br/>``` /api/permission/authorize ``` 有掛驗證 filter, 不帶 token 的情況下打不通，會得到 http code 401, 代表 Unauthorized，反之只要 token 驗證過，就算該 token 不含任何角色，也可以通過，得到回覆
>authorize

<br/>``` /api/administrative/add_acount ``` 有掛驗證 filter, 且限定角色 Administrator，如果 token 裡只有 Teacher 或 Student，就不會通過，會得到 http code 403, 代表 Forbidden -> 注意不是 401 Unauthorized

---

 ## 從 token 取出資料供後續使用

 有時我們也會需要 token 內的資料，例如做新刪修時要要一併存入建立者和修改者，那可以從 token 取得

修改 AuthenticationExtension, 修改傳入 TokenValidationParameters 的參數
 ```csharp
services.AddJwtBearer(x =>
    {
        x.TokenValidationParameters = new TokenValidationParameters
        {
            // 透過這項宣告，就可以從 "sub" 取值並設定給 User.Identity.Name
            NameClaimType = ClaimTypes.NameIdentifier,
            IssuerSigningKey = new SymmetricSecurityKey(key),
            ValidateIssuer = true,
            ValidIssuer = authSetting.Issuer,
            ValidateAudience = true,
            ValidAudience = authSetting.Audience,
            ValidateLifetime = true
        };
    });

```

<br/>加入 UserInfo model
```csharp
public class UserInfo
{
    public string UserId { get; set; }
    public string DisplayName { get; set; }
    public string Email { get; set; }
    public List<string> Roles { get; set; }
}
```

<br/>加入 FetchUserInfoAttribute，解析 token 並將資料放進 HttpContext.Items，如此就可供同個 request 做後續使用
```csharp
public class FetchUserInfoAttribute : ActionFilterAttribute
{
    /// <inheritdoc />
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.HttpContext.User.Identity.IsAuthenticated)
        {
            return;
        }
        
        var userInfo = new UserInfo()
        {
            UserId = context.HttpContext.User.Identity?.Name,
            DisplayName = context.HttpContext.User.Claims.Where(c => c.Type == "display_name").FirstOrDefault()?.Value,
            Email = context.HttpContext.User.Claims.Where(c => c.Type == ClaimTypes.Email).FirstOrDefault()?.Value,
            Roles = context.HttpContext.User.Claims.Where(c => c.Type == ClaimTypes.Role).Select(c => c.Value).ToList()
        };
        context.HttpContext.Items["user_info"] = userInfo;
    }
}
```

<br/>在 Action 掛上 FetchUserInfoAttribute，那麼在 Action 內可直接取得 UserInfo
```csharp
[ApiController]
[Route("api/[controller]")]
public class CourseController : Controller
{
    [Authorize(Roles = "Administrator,Teacher")]
    [HttpPost, Route("set_course")]
    [FetchUserInfo]
    public IActionResult SetCourse([FromBody] Course course)
    {
        var userInfo = HttpContext.Items["user_info"] as UserInfo;
        var courseService = new CourseService();
        courseService.Add(userInfo, course);

        return Ok($"{nameof(SetCourse)} OK");
    }
}
```

---

參考自
- [Working with JWT Tokens In .NET](https://www.codeguru.com/dotnet/jwt-token-dot-net/)
- [Role based JWT Tokens in ASP.NET Core APIs](https://weblog.west-wind.com/posts/2021/Mar/09/Role-based-JWT-Tokens-in-ASPNET-Core#claims-and-roles-roles-roles)