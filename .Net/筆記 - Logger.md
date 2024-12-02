# Logger

## 預設ILogger

### 使用方式

* Custom Property + Message Template
  * Event Id: 作為Custom Property - EventId
  * Message template
    * 用{}括號起來命名，並在後面Input Values，可直接置換為Values
  * Custom Property
    * 同上，此類Format的Input Values，會在Transaction Log顯示為"Custom Properties"

```C#
public class LogSample
{
    private readonly _logger;
    public LogSample(ILogger<LogSample> logger)
    {
        _logger = logger;
    }

    public RunLog()
    {
        _logger.LogInformation(
            eventId: 11,
            message: "Test template. Id: {Id}, Name: {Name}", 
            "0", "timm");
    }
}
```

### Scope (圈定Operation)

* 使用CreateScope讓同一批Log能被同一組Operation ID分組

```C#
using (_logger.BeginScope(new { OperationId = aiOperation.Telemetry.Context.Operation.Id }))
{
    _logger.LogInformation("Start");
    // ...
}
```

* 搭配Application Insights的話，可以共享Operation ID

```C#
using (var aiOperation = _telemetryClient.StartOperation<RequestTelemetry>("Custom Request"))
using (_logger.BeginScope(new { OperationId = aiOperation.Telemetry.Context.Operation.Id }))
{
    _logger.LogInformation("Start");
    // ...
    _telemetryClient.TrackEvent("Complete");
}
```

### Generic Type

* `ILogger<ClassType>`的型別會反映在Log資訊當中的`SourceContext`
* 繼承問題: Bass class vs Derived Class

```C#
public class DerivedClass : BaseClass
{
    public DerivedClass(ILogger<DerivedClass> logger)
        : base(logger)
    {

    }
}

public class BaseClass
{
    public BaseClass(ILogger<BaseClass> logger)
    {
        // 這樣會有型別轉換錯誤
    }

    public BaseClass(ILogger logger)
    {
        // Base Class不指定Generic型別即可使用繼承類別的型別
    }
}
```

