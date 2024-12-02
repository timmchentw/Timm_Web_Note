- Serilog
  - 特點
    - 社群強大、有很多Sink支援
    - 可與.Net Core整合

- High performance logger
  - 特點
    - 算是對比Serilog, NLog的官方套件 (.Net 6以上支援)
    - 可達成Logger function強型別輸入與extension列表的功能
    - 可利用Static partial function搭配Attribute
  - 參考文章
    - [High Performance Logging](https://learn.microsoft.com/en-us/dotnet/core/extensions/high-performance-logging)
    - [Logger Message Generator](https://learn.microsoft.com/en-us/dotnet/core/extensions/logger-message-generator)
  - 範例用法
    - 註冊
  
    ```C#
    readonly file record struct SampleObject { }

    public static partial class Log
    {
        [LoggerMessage(EventId = 23, Message = "{Name} lives in {City}.")]
        public static partial void PlaceOfResidence(
            this ILogger logger,
            LogLevel logLevel,
            string name,
            string city);
    }
    
    public class Example
    {
        public void Execute()
        {
            using ILoggerFactory loggerFactory = LoggerFactory.Create(
                builder =>
                builder.AddJsonConsole(
                    options =>
                    options.JsonWriterOptions = new JsonWriterOptions()
                    {
                        Indented = true
                    }));

            ILogger<SampleObject> logger = loggerFactory.CreateLogger<SampleObject>();
        }
    }
    ```

    - 使用

    ```C#
    logger.PlaceOfResidence(logLevel: LogLevel.Information, name: "Liana", city: "Seattle");
    ```