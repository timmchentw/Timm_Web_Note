# Console Job優化管理
#### 目的
* 整合Environment，允許Job根據環境變數切換行為，並可於Debug時隨時切換
* 集中App Config，便於統一更新與佈署
* 整合Logger，便於紀錄Event
* 集中跨專案邏輯，便於管理

#### 方法
1. 使用與.Net Core Website相同之HostingBuilder建置
2. 設定Environment <br>
  2-1. [Job] 設定launchSettings.json (Debug用) <br>
  2-2. [Builder] 指定Environment Variables Prefix <br>
  2-3. [Server] 設定Deploy環境變數 <br>
    (a) IIS須手動設定Environment Variables、或於Pipeline中指定 <br>
    (b) Web App則需設定Environment Variable
3. 設定Shared Config (參考[出處](https://andrewlock.net/sharing-appsettings-json-configuration-files-between-projects-in-asp-net-core/)) <br>
  3-1. [Shared Project] 建立sharedsettings.json <br>
  3-2. [Shared Project] 設定Always Copy (將檔案複製到Job Assembly Directory) <br>
  3-3. [Builder] 設定config json讀取順序 <br>
4. 設定Logger <br>
  4-1 [Builder] 安裝Serilog Package <br>
  4-2 [Builder] 設定Serilog為預設Logger <br>
  4-3 [Builder] 依個人喜好新增Sink管道 (如App Insights, Email, ...)
5. 建立JobService，用於統一對Exception呼叫Logger
```C#
public class JobSerive
{
    private readonly ILogger<JobSerive> _logger;

    public void JobSerive(JobSerive logger)
    {
        _logger = logger;
    }
    
    public void Run()
    {
        try
        {
            this.JobAction();
        }
        catch (Exception ex)
        {
            _logger.Error(ex);
        }
    }
    
    public virtual void JobAction()
    {
        // Job Logic...
    }
}
```
6. 註冊JobService
7. 執行: Get Service & Run


#### 優化內容
* 整合Azure Application Insights，利於分析Job運作效能