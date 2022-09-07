# .Net Core

## 架構筆記

1. 如果不使用DI相依性注入，有在Constructer New出來使用的class，在結束使用後記憶體將暫時不會釋出，尤其連線相關的variable，因此最好手動dispose
ps. Controller呼叫new service，new service時也會new DB connection，如果request完service並沒有dispose connection就會出現問題 (unit of work會出現、但不使用unit of work則應該每次呼叫DAL時都會using connection -> using完會自動dispose) </br>
2. Unit of work為集中connection的方法，將各自獨立的repository共用SqlConnection，並使用SqlTransaction將各次的SQL指令作為一個單位，如果該單位沒有成功Save change，則所有的改動都可以還原到最開始的狀態 (rollback) </br>
3. 程式架構 View(前端) -> Controller(資料轉換、方法呼叫) -> Service Layer(邏輯處理、呼叫資料源) -> Data Access Layer(控制資料庫、API等連線)，各層間不可以反向引用(但可反向依賴interface)，達成封裝性的要求，以避免耦合過多，如要跨層使用方法可用static方法寫成Utility共用 (如Enum、Util方法等) </br>
4. 如物件導向有類似功能但又不盡相同的class，可將共有的變數、方法提升為base class，再利用繼承擴充base class，擴充方法如果都差不多則用Abstract, Interface去規定名稱、型別等，以避免類似的class中存在大量重複的code、類似方法規格不統一等

## Program & Startup

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

#### Action Filter註冊

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

#### Authorize

實作IAuthorizationFilter

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


## WCF Web Service

