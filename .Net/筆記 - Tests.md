# 自動化測試

## MSTest (V2)

### Lifecycle

* ClassInitialize (Static)
* TestInitialize (Non-Static)
* TestMethod (Non-Static)
* TestCleanup (Non-Static)
* ClassCleanup (Static)

### Test methods

* TestMethod
* DataRow
* DynamicData

```C#
[TestClass]
public class MyTests
{
    private static IHost _host;
    private static IServiceScope _serviceScope;
    private static IService _service;

    // 在所有Test之前 (必須為Static)
    [ClassInitialize]
    public static void Initialize(TestContext testContext)
    {
        // Set Environment
        Environment.SetEnvironmentVariable("ASPNETCORE_ENVIRONMENT", testContext.DeploymentDirectory.ToLower().Contains("debug") ? Environments.Development : Environments.Staging);

        // Dependency Injection
        var builder = WebApplication.CreateBuilder();
        builder.Services.AddSingleton(builder.Configuration.GetSection("MyConfig").Get<MyConfig>());    // Configuration in appsettings.json
        builder.Services.AddScoped<IService, MyService>();
        _host = builder.Build();
    }

    // 在每次Test之前
    [TestInitialize]
    public void TestInit()
    {
        _serviceScope = _host.Services.CreateScope();   // Create scope for each test
        _service = _serviceScope.ServiceProvider.GetRequiredService<IService>();    // Get service of testing
    }

    // 簡單Cases (僅限Const參數)
    [DataTestMethod]
    [DataRow(true)]
    [DataRow(false)]
    public void MyMethod1Test(bool isChecked)
    {
        var result = _testClass.MyMethod1();
        Assert.IsTrue(result);
    }

    // 複雜Cases (非Const參數)
    private static IEnumerable<object[]> MyMethod2TestCases
    {
        get
        {
            yield return new object[] { "2021", "output1" }; 
            yield return new object[] { "2022", "output2" }; 
        }
    }
    [DataTestMethod]
    [DynamicData(nameof(MyTest1Cases), DynamicDataSourceType.Property)]
    public void MyMethod2Test(string input, string expect)
    {
        var result = _testClass.MyMethod2(input);
        Assert.AreEqual(result, expect);
    }

    // 在每次Test之後
    [TestCleanup]
    public void Cleanup()
    {
        _serviceScope.Dispose();    // Clear service scope of current test
    }

    // 在所有Test結束之後 (必須為Static)
    [ClassCleanup]
    public static void Clean(TestContext testContext)
    {
        _host.Dispose();   // Clear app host
    }
}
```

### DI註冊 (參考[黑大文章](https://blog.darkthread.net/blog/aspnetcore-efcore-unitest/))

```C#
```C#
[TestClass]
public class MyTests
{
    private IWebHost _webHost;
    private T GetService<T>()
    {
        var scope = _webHost.Services.CreateScope();
        return scope.ServiceProvider.GetRequiredService<T>();
    }

    [ClassInitialize]
    public static void Init(TestContext testContext)
    {
        _webHost = WebHost.CreateDefaultBuilder()
            .UseStartup<Startup>()
            .Build();
    }

    [TestInitialize]
    public void Initialize()
    {
        
    }
}
```

