# 在 .Net Core WebApi 使用 JWT 驗證

使用平台: .Net Core 6.0

使用套件:
- Microsoft.AspNetCore.Authentication.JwtBearer

todo
- 如何自訂驗證 token, 例如 bearer 帶芝麻開門也可以過, 方便開發時測試
- 增加自訂 handler
- 如何從 token 取出角色, userId 供後續使用
---

## 陽春版

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

<br/>增加 JwtHelper, 負責生成 jwt token
```csharp
public class JwtHelper
{
    private readonly AuthSetting _authSetting;

    public JwtHelper(IConfiguration config)
    {
        _authSetting = config.GetSection("AuthSetting").Get<AuthSetting>();
    }

    public string GenerateSecurityToken(string email)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_authSetting.Secret);
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new[]
            {
                new Claim(JwtRegisteredClaimNames.Iss, _authSetting.Issuer),
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
                new Claim(ClaimTypes.Email, email)
            }),
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
                    ValidateAudience = true,
                    ValidIssuer = authSetting.Issuer,
                    ValidAudience = authSetting.Audience
                };
            });

        return services;
    }
}
```

<br/>在 program.cs 加上指定用 AuthenticationExtension
```csharp
var configuration = builder.Configuration;

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddTokenAuthentication(configuration);
```

<br/>呼叫 app.UseAuthentication();
```csharp
app.UseHttpsRedirection();

// 注意順序很重要
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

<br/>在 swagger 使用 jwt token 驗證
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
        var jwtHelper = new JwtTokenHelper(_config);
        var token = jwtHelper.GenerateSecurityToken("fake@email.com");  
        return Ok(token);
    }
}
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

增加 Controller, 一個 Action 要驗證, 一個 Action 不用
```csharp
[ApiController]
[Route("api/[controller]")]
public class PermissionController : Controller
{
    [AllowAnonymous]
    [HttpGet, Route("anonymous")]
    public IActionResult Anonymous()
    {
        return Ok("anonymous");
    }
    
    [Authorize]
    [HttpGet, Route("authorize")]
    public IActionResult Authorize()
    {
        return Ok("authorize");
    }
}
```


我們直接跑 debug mode, 用 swagger 打 ```/api/Permission/anonymous```, 會得到回覆
>anonymous

用 swagger 打 ```/api/Permission/authorize```, 會得到 http code 401, 代表 Unauthorized


將取得的 token 設定到 swaggger, 再去打 ```/api/Permission/authorize``` 就不會被擋住，會得到
>authorize

---

## 加入角色

加入角色 enum
```csharp
public enum MyRole
{
    Administrator = 0,
    Teacher = 1,
    Student = 2,
}
```

<br/>增加 JwtHelper
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
            new Claim(ClaimTypes.Email, email)
        };
        // 塞入角色
        claims.AddRange(roles.Select(r => new Claim(ClaimTypes.Role, r.ToString())));
        
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

在 PermissionController 加入以下 Action
```csharp
    [Authorize(Roles = "Administrator")]
    [HttpGet, Route("administrator")]
    public IActionResult Administrator()
    {
        return Ok("authorize administrator");
    }
    
    [Authorize(Roles = "Administrator,Teacher,Student")]
    [HttpGet, Route("user")]
    public IActionResult User()
    {
        return Ok("authorize user");
    }
    
    [Authorize(Roles = "Administrator,Teacher")]
    [HttpGet, Route("teacher")]
    public IActionResult Teacher()
    {
        return Ok("authorize teacher");
    }
```

如果 token 的角色有 Teacher 和 Student, 用 swagger 打 ```/api/Permission/administrator```, 會得到 http code 403, 代表 Forbidden

如果 token 的角色有 Teacher 和 Student, 用 swagger 打 ```/api/Permission/user```, 會得到
>authorize user

解出來的 token 範例
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

---

參考自
- [Working with JWT Tokens In .NET](https://www.codeguru.com/dotnet/jwt-token-dot-net/)
- [Role based JWT Tokens in ASP.NET Core APIs](https://weblog.west-wind.com/posts/2021/Mar/09/Role-based-JWT-Tokens-in-ASPNET-Core#claims-and-roles-roles-roles)