# Azure Web App

### [Environment Variables]
![1.png](images/web_app/1.png "")

### [Diagnose]

可根據Azure建議的方式變更安全性與穩定性設定

* Application Logs：提供Deployment, 執行時的Log
* Container Crash：Pipeline Debug時可用
    ![4.png](images/web_app/4.png "")
    ![5.png](images/web_app/5.png "")
* Http Server Error：提供Request分析，如Method, Uri, Count等等
    ![2.png](images/web_app/2.png "")
* Monitoring→Health check：定期呼叫Request確認運作狀況 (如為避免App Insights過多Log，可設定Path為/home/login)
    ![3.png](images/web_app/3.png "")


### [Deployment Slots]

