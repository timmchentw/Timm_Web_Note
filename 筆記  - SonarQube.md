# SonarQube
## 來源碼分析 (漏洞檢查、語法建議)
### 安裝
#### A.	安裝本體
1. 下載[SonarQube](https://www.sonarqube.org/downloads/)專案 (Community)<br>
2. 設定：解壓縮Zip，並更改..\conf\sonar.properties，把sonar.search.port設為0 (預設9001 port可能會被占用)，並把該行的#拿掉
![image]()<br>
3. 執行..\bin\windows-x86-64\StartSonar.bat (關閉此CMD則會把應用程式關掉)<br>
4. 開啟Chrome，輸入http://localhost:9000/ ，即可進入到SonarQube網頁<br>
5. 建立分析專案：到Administrator頁籤 → Prjects → Management → Create project<br>
6. 輸入token name名稱，取得實際token <br>
7. 跟著步驟進到.net版本的[sonar scanner頁面](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/)，照步驟安裝<br>
8. Sonar scanner必要環境：[Build tools for Visual Studio](https://visualstudio.microsoft.com/zh-hant/downloads/) (從VS安裝程式安裝)<br>
9. 下載Sonar scanner的MSBuild Zip檔，解壓縮後，把主要資料夾路徑加入system環境變數
10. 更改sonar scanner foler檔案: ../SonarQube.Analysis.xml，把裡面的sonar.login更改為剛才取得的token<br>
11. 跟著SonarQube本體的最後步驟，在欲分析的.sln專案目錄下，在url列輸入cmd，並輸入從SonarQube取得的以下指令：
```
SonarScanner.MSBuild.exe begin /k:"??????" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="!!!!!!!!!!!!!!!!!!!!"
"C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\MSBuild\Current\Bin\"MsBuild.exe /t:Rebuild
SonarScanner.MSBuild.exe end /d:sonar.login="!!!!!!!!!!!!!!!!!!!!"
```
(????會根據專案名稱而變、!!!!!是你自己的token、MsBuild.exe前面要加上實際路徑才能找到該exe)<br>
12. 完成!SonarQube會顯示分析結果!
![image](https://raw.githubusercontent.com/timmchentw/TimmWebPractice/master/pics/SonarQube%20sendEmailJob.png)
