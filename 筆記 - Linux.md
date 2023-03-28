# Linux筆記

## Nginx

## 佈署Web Service

### .Net Core Krestral

* Nginx反向代理至Krestal：
    * 預設 - `/etc/nginx/sites-available/default.conf`
    * 每個網站各自設定 - `/etc/nginx/conf.d/XXX.conf`

    ```conf
    server {
        listen        80;
        server_name   example.com *.example.com;
        location / {
            proxy_pass         http://127.0.0.1:5000;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
    ```

    * 測試conf運作：`sudo nginx -t`
    * 套用conf設定：`sudo nginx -s reload`

    ※ Nginx Log存在/var/log/nginx/，檔案包括access.log、error.log

* 網站設定檔(使用systemd服務)：</br>
  * 檔案: `/etc/systemd/system/kestrel-XXX.service`
  
    ```systemd
    [Unit]
    Description=Example .NET Web API App running on Ubuntu

    [Service]
    WorkingDirectory=/var/www/XXX
    ExecStart=/usr/bin/dotnet /var/www/XXX/XXX.dll
    Restart=always
    # Restart service after 10 seconds if the dotnet service crashes:
    RestartSec=10
    KillSignal=SIGINT
    SyslogIdentifier=dotnet-example
    User=www-data
    Environment=ASPNETCORE_ENVIRONMENT=Production
    Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

    [Install]
    WantedBy=multi-user.target
    ```

  * 上述dll檔案務必注意大小寫!
  * 事先加入新user for service & deploy </br>
  `sudo groupadd www-data` </br>
  `sudo useradd -g www-data www-data` </br>
  `sudo chmod g+w -R /var/www/XXX` </br>

  * 啟用網站： </br>
  `sudo systemctl enable kestrel-XXX.service` </br>
  `sudo systemctl start kestrel-XXX.service` </br>
  `sudo systemctl status kestrel-XXX.service`

  * 查看執行LOG </br>
  `sudo journalctl -fu kestrel-XXX.service`

  * 重新執行 </br>
  `sudo systemctl restart kestrel-XXX.service`

  * 關閉 </br>
  `sudo systemctl stop kestrel-XXX.service`

