# .Net Core

## 架構筆記

1. 如果不使用DI相依性注入，有在Constructer New出來使用的class，在結束使用後記憶體將暫時不會釋出，尤其連線相關的variable，因此最好手動dispose
ps. Controller呼叫new service，new service時也會new DB connection，如果request完service並沒有dispose connection就會出現問題 (unit of work會出現、但不使用unit of work則應該每次呼叫DAL時都會using connection -> using完會自動dispose) </br>
2. Unit of work為集中connection的方法，將各自獨立的repository共用SqlConnection，並使用SqlTransaction將各次的SQL指令作為一個單位，如果該單位沒有成功Save change，則所有的改動都可以還原到最開始的狀態 (rollback) </br>
3. 程式架構 View(前端) -> Controller(資料轉換、方法呼叫) -> Service Layer(邏輯處理、呼叫資料源) -> Data Access Layer(控制資料庫、API等連線)，各層間不可以反向引用(但可反向依賴interface)，達成封裝性的要求，以避免耦合過多，如要跨層使用方法可用static方法寫成Utility共用 (如Enum、Util方法等) </br>
4. 如物件導向有類似功能但又不盡相同的class，可將共有的變數、方法提升為base class，再利用繼承擴充base class，擴充方法如果都差不多則用Abstract, Interface去規定名稱、型別等，以避免類似的class中存在大量重複的code、類似方法規格不統一等

## Program & Startup

### Authentication

* Authentication為驗證User登入、身分是否合法，與Authorization不同，另一者主要作為Role assign
* 在Startup的部分，設定AddAuthentication middleware

```C#
// .Net 6 Program.cs
var builder = WebApplication.CreateBuilder(args);
// ...
builder.Services.AddScoped<MyCookieAuthenticationEvents>(); // 註冊客製化Auth事件(登入轉址、登出等)
// ...
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.ExpireTimeSpan = TimeSpan.FromHours(12);
        options.SlidingExpiration = false;
        options.EventsType = typeof(MyCookieAuthenticationEvents);
        options.AccessDeniedPath = "/Home/AccessDeny";
        options.LoginPath = $"/Home/Login";
    });
// ...
var app = builder.Build();
app.UseAuthentication();
```

* 自定義CookieAuthenticationEvents middleware (記得在上方EventsType註冊)
* 可客製如取得當下的url、domain轉換等等

```C#
public class MyCookieAuthenticationEvents : CookieAuthenticationEvents
{
    public MyCookieAuthenticationEvents()
    {
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        if (userPrincipal == null ||
            !string.IsNullOrEmpty(userPrincipal.Claims.Where(x => x.Type == ClaimTypes.NameIdentifier).FirstOrDefault()?.Value))
        {
            // 設定登出條件 (看登入時claim如何設定)
            context.RejectPrincipal();
            await context.HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            return;
        }
    }

    public override Task RedirectToLogin(RedirectContext<CookieAuthenticationOptions> context)
    {
        // 可設定domain轉換
        string host = context.Request.Host.Value;
        if (context.Request.Host.Host.Equals("mywebsitesample.azurewebsites.net", StringComparison.OrdinalIgnoreCase))
        {
            host = "my.domain.com";
        }

        // 取得現在的url，並在轉址時附上為query string
        var currentUrl = $"{context.Request.Scheme}://{host}{context.Request.PathBase}{context.Request.Path}{context.Request.QueryString}";
        string loginUrl = $"https://{host}{context.Options.LoginPath}";

        var redirectUrlWithReturnUrl = QueryHelpers.AddQueryString(loginUrl, "returnUrl", currentUrl);
        context.Response.Redirect(redirectUrlWithReturnUrl);

        return Task.CompletedTask;
    }
}
```

#### Shared cookie (Single sign-on)

* 注意要設定cookie name & path (不同path的cookie不互通)
* 也要設定data protection (用於在不同Websites間解密cookie內容)
* 指定SameSite的cookie存取嚴格程度(strict, lax, none...)
* 範例參考[官方文件](https://learn.microsoft.com/zh-cn/aspnet/core/security/cookie-sharing?view=aspnetcore-8.0)、[cookie嚴格程度說明](https://learn.microsoft.com/en-us/aspnet/core/security/samesite?view=aspnetcore-8.0)、[Data Protection設定](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0)

```C#
// .Net 6 Program.cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        // 這塊跟上方一樣
        options.ExpireTimeSpan = TimeSpan.FromHours(24);
        options.SlidingExpiration = false;
        options.EventsType = typeof(MyCookieAuthenticationEvents);
        options.AccessDeniedPath = "/Home/AccessDeny";
        options.LoginPath = "/Home/Login";

        // 這塊設定共享cookie
        options.Cookie.SameSite = SameSiteMode.Strict;  // 可設定嚴格程度
        options.Cookie.Name = ".AspNet.SharedCookie";
        options.Cookie.Path = "/";  // 避免有不同basePath時，cookie可能會存在不同路徑 (如/v1, /v2會被視為不同存在)
        options.DataProtectionProvider = DataProtectionProvider.Create(
            new DirectoryInfo(
                (RuntimeInformation.IsOSPlatform(OSPlatform.Windows)) ?
                    @"C:\inetpub\wwwroot\MyWebsite\Keys\"
                    : "/var/www/MyWebsite/Keys/"));  // key加密檔案存在server端，host端必須要access到同一個檔案才行(即各個websites要放在同一個file system裡面)
    });
```

* 跨Server需要使用到KeyVault、或是DB，可參考[官方文件說明](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0#protectkeyswithazurekeyvault)，以下為dbcontext的用法 (MyDbContext必須實作`IDataProtectionKeyContext`)

```C#
// .Net 6 Program.cs
using Microsoft.AspNetCore.DataProtection;
// ...
builder.Services.AddDataProtection()
    .SetApplicationName("AllWebsiteShouldUseSameName")
    .PersistKeysToDbContext<MyDbContext>();
// Cookie那邊不需特定設定，記得上面區塊的'options.DataProtectionProvider'就不用特別設定了
```

* 想瞭解.Net Core底層如何判定cookie是否合法，可參考以下部分進行debug (中斷點可下在上方自行覆寫的MyCookieAuthenticationEvents當中)
  * HandleAuthenticateAsync
  * ReadCookieTicket
  * HandleChallengeAsync
  * 注意：如果無論如何驗證都過了還是無法通過authorize，可檢查一下middleware設置的`UseAuthentication`必須放在`UseAuthorization`前面
  ![image](images/.Net%20Core/9.png) </br>
  * F12→Application查看Cookie資料
    ![image](images/.Net%20Core/10.png) </br>

#### Redirect to login

* login get提供return url (由前端傳到post)
* 

```C#
public class HomeController
{
    private AccountService _accountService;
    private HomeController _logger;

    public HomeController(IHttpContextAccessor httpContextAccessor, AccountService accountService, ILogger<HomeController> logger)
    {
        _accountService = accountService;
        _logger = logger;
    }

    // login page記得允許匿名存取
    [AllowAnonymous]
    public IActionResult Login(string? returnUrl)
    {
        // 由前一個頁面提供Return URL (參考前面MyCookieAuthenticationEvents.RedirectToLogin)
        ViewBag["ReturnUrl"] = returnUrl;
        return View();
    }

    // login page記得允許匿名存取
    [HttpPost]
    [AllowAnonymous]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Login(LoginViewModel model)
    {
        if (ModelState.IsValid)
        {
            // 自行驗證user合法性(如DB, API資料驗證)
            var loginTicket = await _accountService.Login(new LoginDto()
            {
                Email = model.Email,
                Password = model.Password,
                ClientIp = HttpContext.Connection.RemoteIpAddress?.ToString() ?? ""
            });

            if (loginTicket.Success && loginTicket.Data != null)
            {
                var account = loginTicket.Data;

                // Identity日後可從HttpContext當中取得登入時的資訊
                var claims = new List<Claim>()
                {
                    new Claim(ClaimTypes.NameIdentifier, account.Id.ToString()),
                    new Claim(ClaimTypes.Name, string.IsNullOrEmpty(account.Name) ? String.Empty : account.Name),
                    new Claim(ClaimTypes.Role, JsonConvert.SerializeObject(user.Roles)),
                };

                var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

                // 設定cookie sign in
                await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, new ClaimsPrincipal(claimsIdentity),
                    new AuthenticationProperties()
                    {
                        ExpiresUtc = DateTime.UtcNow.AddHours(12),
                        IsPersistent = true,
                    });

                _logger.LogInformation("User logged in. Name: {Name}, Time: {Time} .", account.Name, DateTime.Now.ToString());

                // Return url前安全檢查
                if (IsValidReturnUrl(model.ReturnUrl))
                {
                    return Redirect(model.ReturnUrl);
                }
                else
                {
                    return RedirectToAction("Index");
                }
            }
            else
            {
                ViewBag["ResponseCode"] = loginTicket.Code;
                ViewBag["ResponseMessage"] = loginTicket.Message;
            }
        }
        return View();
    }

    [AllowAnonymous]
    public async Task<IActionResult> LogOut(string? returnUrl)
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);

        // Return url前安全檢查
        if (IsValidReturnUrl(returnUrl))
        {
            return Redirect(returnUrl);
        }
        else
        {
            return RedirectToAction("Index");
        }
    }

    private bool IsValidReturnUrl(string returnUrl)
    {
        bool isValid = false;

        if (!string.IsNullOrEmpty(returnUrl))
        {
            var requestHost = Accessor.HttpContext?.Request?.Headers["X-Forwarded-Host"].FirstOrDefault()   // DNS設定的公開domain
                ?? Accessor.HttpContext?.Request?.Host.Host;

            // 比較Return url Domain與目前站台是否相同
            if (Uri.TryCreate(returnUrl, UriKind.Absolute, out Uri? uriResult))
            {
                isValid = string.Equals(uriResult.Host, requestHost, StringComparison.OrdinalIgnoreCase);
            }
            else
            {
                _logger.LogInformation("Parse returnUrl failed. Current domain: {CurrentDomain}; Return url: {ReturnUrl}", requestHost, returnUrl);
            }
        }
        return isValid;
    }
}
```

#### Identity

* 官方[非Identity架構方法指南](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie?view=aspnetcore-8.0)
* Claim & Identity說明見[ASP.NET Core Authentication系列（一）理解Claim, ClaimsIdentity, ClaimsPrincipal](https://www.cnblogs.com/liang24/p/13910368.html)
  * HttpContextAccessor可取得當時存的Identity

    ```C#
    public class MyService
    {
        protected int? UserId
        {
            get
            {
                return Convert.ToInt32(_httpContext.HttpContext?.User.FindFirstValue(ClaimTypes.NameIdentifier));
            }
        }

        protected IHttpContextAccessor _httpContext { get; }
        public BaseService(IHttpContextAccessor httpContextAccessor)
        {
            _httpContext = httpContextAccessor;
        }
    }
    ```


* 有設定才能在Controller上設置[Authorize]來驗證User

### Route

* 運作環境有route prefix時，可在Map Route前呼叫UsePathBase (務必要放在所有route設定前)，這樣有無prefix都可以正常呼叫
* 注意login path也需要加上prefix才能在登入時導到正確的路徑

```C#
// .Net 6 Program.cs
app.UsePathBase("/v2"); // 這邊設定prefix

// ...
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    // 這邊要注意將原本的index page轉為有prefix的index page，避免cookie無法辨別
    endpoints.MapGet("/", context =>
    {
        context.Response.Redirect($"{RouteResource.BackendRoutePrefix}/Home/Index");
        return Task.CompletedTask;
    });
    //...
});
// Cookie驗證的部分，要注意login path也要包含basepath (或是於自行施作CookieAuthenticationEvents時要把prefix也包含進去Url)
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/v2" + "/Home/Login";
    });
```

### Dependency Injection

#### 註冊服務

* AddSingleton
* AddScoped
* AddTransient

#### 取得服務

* Host (IHost, IWebHost...) instance

    ```C#
    app.ApplicationServices.GetService<IMyService1>();  // Return null if no service exisits
    app.ApplicationServices.GetRequiredService<IMyService2>();  // Throw excetion if no service exists
    ```

    ※ 此方法僅適用singleton service，建議scoped service使用`services.CreateScope`，再使用`scope.ServiceProvider.GetRequiredService`來取得</br></br>

* Registered service

    ```C#
    public MyService2 : IMyService2
    {
        private readonly IMyService1 _myService1;
        public MyService2(IMyService1 myService1)
        {
            // 建議使用此方法，而不是下方
            _myService1 = myService1;
        }

        public MyService2(IServiceProvider provider)
        {
            // 此方法雖然簡潔，但是在Build host時無法檢查instance是否為null
            // 而且會迫使所有Service在每個Request都New Instance (不管有沒有用到)，記憶體堪憂!
            _myService1 = provider.GetRequiredService<IService1>();
        }
    }
    ```

### Use app

### Middlewares

#### Middleware註冊

- Middleware是針對HTTP請求的中介層

```C#
// Program.cs (.Net 6)

builder.Services.AddTransient<MyMiddleware>();
//...
var app = builder.Build();
//...
app.UseMiddleware<MyMiddleware>();
```

```C#
// MyMiddleware.cs

public class MyMiddleware : IMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
         // Wait for next middleware return (The response body will be written)
        await next(context);

        // ...
    }
```

#### Filter註冊

- Action Filter屬於MVC獨有中介層，與Middleware不同
- 可利用Exception, Action & Result Filter捕捉資訊

```C#
// Startup.cs

services.AddControllersWithViews(options =>
{
    options.Filters.Add<ValidateModelFilter>();  // Call ModelState.IsValid for controller
})
```

```C#
// ValidateModelFilter.cs

public class ValidateModelFilter : IActionFilter
{
    public void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.ModelState.IsValid)
        {
            return;
        }

        var validateErrors = context.ModelState.Select(x => x.Value.Errors).Where(y => y.Count > 0);
        string errorMsg = String.Join("; ", validateErrors.Select(x => x.FirstOrDefault().ErrorMessage ?? x.FirstOrDefault().Exception.Message));
        context.Result = new ObjectResult(errorMsg)
        {
            StatusCode = 400
        };
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Do nothing
    }
}
```

##### Authorize Filter

* 設定步驟

  1. 需要在program.cs當中設定middleware `UseAuthorization`
     * 注意!!!!! 一定要放在UseAuthentication後面! 先驗證再授權，否則永遠進不到Authentication middleare!)

        ```C#
        // Program.cs
        app.UseAuthentication();
        app.UseAuthorization(); // <--一定要放在UseAuthentication後面! 先驗證user合法性、再處理授權
        ```

  2. Controller/Action當中加入Attribute `[Authorize]`

        ```C#
        [Authorize] // 擇一放置即可
        public class HomeController : Controller
        {
            [Authorize] // 擇一放置即可
            public IActionResult Index()
            {
                return View();
            }
        }
        ```

  3. `[AllowAnonymous]`會覆蓋過`[Authorize]`：比如controller設定`[Authorize]`，則特定action要跳過授權檢查的話加入`[AllowAnonymous]`即可

        ```C#
        [Authorize] // 全部action套用
        public class HomeController : Controller
        {
            [AllowAnonymous] // 特定action不需要"授權前"的驗證
            public IActionResult Index()
            {
                return View();
            }
        }
        ```


* 客製化實作IAuthorizationFilter

    ```C#
    [AttributeUsage(AttributeTargets.All)]
    public class MyAuthorizeAttribute : Attribute, IAuthorizationFilter
    {
        public void OnAuthorization(AuthorizationFilterContext context)
        {
            var userService = context.HttpContext.RequestServices.GetRequiredService<IUserService>();
            var userId = context.HttpContext.User.FindFirstValue(ClaimTypes.NameIdentifier);
            var controllerName = context.HttpContext.Request.RouteValues["controller"]?.ToString();

            if (!userService.HasPermission(userId, controllerName))
            {
                context.Result = new ForbidResult();
            }
        }
    }
    ```

  * Action指定需要授權的Role

  ```C#

  ```

## Environment

### Environment Variable

#### Set

* Unit test </br>
  ![1.png](images/.Net%20Core/1.png) </br>
* Debug </br>
  ![2.png](images/.Net%20Core/2.png) </br>
  ![3.png](images/.Net%20Core/3.png) </br>
  ![4.png](images/.Net%20Core/4.png) </br>
* IIS </br>
  ![6.png](images/.Net%20Core/6.png) </br>
* Azure Web App </br>
  ![7.png](images/.Net%20Core/7.png) </br>
* Process (PowerShell) </br>
  ![5.png](images/.Net%20Core/5.png) </br>

#### Get

## Frontend

### Tag Helper

- 可設定類似HTML Tag的自定義Element，產生HTML & JS Text
- 設定方式
  - 建立Class, Implement TagHelper & Attribute HtmlTargetElement
- 優點:
  - 強型別物件，方便追蹤
  - 有後端驗證
  - 前後端整合
- 缺點:
  - 部分設定不全，前端工具支援度受限(需要理解整個前端套件的設定)
  - 動態功能支援性受限(Razor只渲染一次)
  - 有時候設定上或可讀性較不直覺
  - 前後端職責混和
- 與前端架構使用: 視情況與習慣使用
- 範例

    ```C#
    // backend
    using Microsoft.AspNetCore.Razor.TagHelpers;

    [HtmlTargetElement("email")]
    public class EmailTagHelper : TagHelper
    {
        public string Address { get; set; }
        public string Display { get; set; }

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            output.TagName = "a"; // 更改tag為<a>
            output.Attributes.SetAttribute("href", $"mailto:{Address}");
            output.Content.SetContent(Display);
        }
    }
    ```

    ```HTML
    <!--.cshtml-->
    @addTagHelper *, YourNamespace

    <!DOCTYPE html>
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Email Tag Helper Example</title>
    </head>
    <body>
        <h1>Email Tag Helper Example</h1>

        <email address="john@example.com" display="Email John"></email>
    </body>
    </html>
    ```

  - 輸出結果範例

    ```HTML
    <a href="mailto:john@example.com">Email John</a>
    ```

## Deployment

### Linux

### 問題排解

* 站台運作後出現HTTP Error 500.30 - ANCM In-Process Start Failure
  * appsettings.json有格式錯誤，移除錯誤即可
  * Visual studio有bug，更新即可
## 傳輸協定

### gRPC

1. 建立 gRPC 服務和客戶端
    1. 步驟一：建立 gRPC 服務
    開啟 Visual Studio 2022，選擇 新建專案。
    搜尋 gRPC，選擇 ASP.NET Core gRPC 服務，然後點擊 下一步。
    在 配置新專案 對話框中，輸入專案名稱（例如 GrpcGreeter），選擇 .NET 8.0 (Long Term Support)，然 後點擊 建立。
    在 Additional information 對話框中，選擇 Create。
    2. 步驟二：編輯 proto 文件
    在專案中，找到 Protos/greet.proto 文件。
    編輯 greet.proto 文件以定義 gRPC 服務和消息。

    ```C#
    syntax = "proto3";

    option csharp_namespace = "GrpcGreeter";

    package greet;

    // The greeting service definition.
    service Greeter {
    // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply);
    }

    // The request message containing the user's name.
    message HelloRequest {
    string name = 1;
    }

    // The response message containing the greetings
    message HelloReply {
    string message = 1;
    }

    ```

   3. 步驟三：實現 gRPC 服務
   在 Services 文件夾中，創建 GreeterService.cs 文件。
   實現 GreeterService 類：
    (Build後會自動生成Model檔案-HelloRequest)

    ```C#
    using Grpc.Core;
    using GrpcGreeter;

    public class GreeterService : Greeter.GreeterBase
    {
        private readonly ILogger<GreeterService> _logger;
        public GreeterService(ILogger<GreeterService> logger)
        {
            _logger = logger;
        }

        public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        {
            return Task.FromResult(new HelloReply
            {
                Message = "Hello " + request.Name
            });
        }
    }
    ```

2. 建立 gRPC 客戶端
   1. 步驟一：建立新的 .NET Console 應用程式
   在 Visual Studio 中，建立新的 .NET Console 應用程式。
   添加 Grpc.Net.Client 和 Google.Protobuf NuGet 套件。
   2. 步驟二：編寫客戶端代碼
   在 Program.cs 文件中，編寫以下代碼：

   ```C#
   using Grpc.Net.Client;
   using GrpcGreeter;
   using System.Threading.Tasks;

   class Program
   {
       static async Task Main(string[] args)
       {
           // The port number must match the port of the gRPC server.
           using var channel = GrpcChannel.ForAddress("https://localhost:5001");
           var client = new Greeter.GreeterClient(channel);
           var reply = await client.SayHelloAsync(new HelloRequest { Name = "World" });
           Console.WriteLine("Greeting: " + reply.Message);
       }
   }
   ```
