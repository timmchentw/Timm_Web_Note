# Application Insights

## Logger

### Website

### Console App

## Filter (Processor)

* 適用於想過濾特定Telemetry參數的情境

    ```C#
    public class CustomTelemetryProcessor : ITelemetryProcessor
    {
        private readonly ITelemetryProcessor _next;

        public CustomTelemetryProcessor(ITelemetryProcessor next)
        {
            this._next = next;
        }

        public void Process(ITelemetry item)
        {
            if (!IsOkToLog(item))
            {
                return; // Skip logging
            }

            this._next.Process(item);
        }

        private bool IsOkToLog(ITelemetry item)
        {
            if (item is RequestTelemetry)
            {
                var requestTelemetry = item as RequestTelemetry;
                if (requestTelemetry.Url.AbsoluteUri.Split('?')[0].EndsWith(".js") ||
                    requestTelemetry.Url.AbsoluteUri.Split('?')[0].EndsWith(".css") ||
                    requestTelemetry.Url.AbsoluteUri.Split('?')[0].EndsWith(".ico"))
                {
                    return false;
                }
            }

            return true;
        }
    }
    ```

## Custom Property (Initializer)

* 適用於想加入Custom Property在Log當中

    ```C#
    public class CustomTelemetryInitializer : ITelemetryInitializer
    {
        private readonly Parameters _parameters;
        private readonly IHostEnvironment _environment;

        // next will point to the next TelemetryProcessor in the chain.
        public CustomTelemetryInitializer(Parameters parameters, IHostEnvironment environment)
        {
            _parameters = parameters;
            _environment = environment;
        }

        public void Initialize(ITelemetry telemetry)
        {
            // 設定Custom Property
            if (!telemetry.Context.GlobalProperties.ContainsKey("AppRunningEnviroment"))
            {
                telemetry.Context.GlobalProperties.Add("AppRunningEnviroment", _environment.EnvironmentName);
            }

            if (!telemetry.Context.GlobalProperties.ContainsKey("Application"))
            {
                telemetry.Context.GlobalProperties.Add("Application", _parameters.ProgramNamespace);
            }
        }
    }
    ```

## Custom Request



## Ajax紀錄!
* 可以記錄Request time, ip, crountry, session...
* 

## 相關資源
[[MSDN] Application Insights for Worker Service applications (non-HTTP applications)](https://docs.microsoft.com/en-us/azure/azure-monitor/app/worker-service#aspnet-core-background-tasks-with-hosted-services) <br>
[[Peter Bons] Monitoring non-web apps using Azure Application Insights](https://dev.to/expecho/monitoring-non-web-apps-using-azure-application-insights-part-2-basic-instrumentation-2fcj) <br>
[[TheCodeBuzz] Waiting for the Host to be disposed. Ensure all IHost instances are wrapped in using block](https://www.thecodebuzz.com/waiting-for-host-disposed-ensure-ihost-instances-wrapped-in-using-blocks/) <br>
[[stackoverflow] Application Insights Developer Mode not working in ASP.NET Core 3.1](https://stackoverflow.com/questions/62856487/application-insights-developer-mode-not-working-in-asp-net-core-3-1)
[[stackoverflow] TelemetryClient does not send any data unless Flush is called](https://stackoverflow.com/questions/30904930/telemetryclient-does-not-send-any-data-unless-flush-is-called)