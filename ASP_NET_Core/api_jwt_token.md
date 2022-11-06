# 在 .Net Core WebApi 使用 JWT 驗證

使用平台: .Net Core 6.0

使用套件:
- Microsoft.AspNetCore.Authentication.JwtBearer
- System.IdentityModel.Tokens.Jwt

todo
- 如何塞角色、權限到 token
- 設定 iss, aud 的不同值
- 如何自訂驗證 token, 例如 bearer 帶芝麻開門也可以過, 方便開發時測試
- 增加自訂 handler
---

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

<br/>增加 JwtService, 負責生成 jwt token
```csharp
public class JwtService
{
    private string mysecret;
    private string myexpDate;

    public JwtService(IConfiguration config)
    {
        mysecret = config.GetSection("JwtConfig").GetSection("secret").Value;
        myexpDate = config.GetSection("JwtConfig").GetSection("expirationInMinutes").Value;
    }

    public string GenerateSecurityToken(string email)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(mysecret);
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new[]
            {
                new Claim(JwtRegisteredClaimNames.Iss, "localhost"),
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
                new Claim(ClaimTypes.Email, email)
            }),
            Audience = "localhost",
            Expires = DateTime.UtcNow.AddMinutes(double.Parse(myexpDate)),
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
  "JwtConfig": {
    "secret": "RFv7DrqznYL6nv7DrqdfrenQYO9JxIsWdcjnQYL6nu09f",
    "expirationInMinutes": 1440
  }
}

```

<br/>增加 JwtService, 負責驗證 jwt token
```csharp
public static class AuthenticationExtension
{
    public static IServiceCollection AddTokenAuthentication(this IServiceCollection services, IConfiguration config)
    {
        var secret = config.GetSection("JwtConfig").GetSection("secret").Value;

        var key = Encoding.ASCII.GetBytes(secret);
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
                    ValidIssuer = "localhost",
                    ValidAudience = "localhost"
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

<br/>增加登入的 action , 將取得的 token 設定到 swaggger, 就可以用 swagger 去打最一開始的 action 了
```
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
        var jwt = new JwtService(_config);
        var token = jwt.GenerateSecurityToken("fake@email.com");  
        return Ok(token);
    }
}
```

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

參考自[Working with JWT Tokens In .NET](https://www.codeguru.com/dotnet/jwt-token-dot-net/)