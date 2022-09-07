## Bot framework

### 必要環境
* Visual studio 2017 以上         <br>
* Bot Framework SDK V4 (nuGet)         <br>
https://github.com/microsoft/botframework-sdk         <br>
* bot framework template for VS (extension)         <br>
https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4         <br>
* 測試工具 (setUp)         <br>
https://github.com/Microsoft/BotFramework-Emulator/         <br>

### 建置過程
1. 建置Bot專案 (Visual C# → Bot framework → Echo Bot (v4))      <br>
2. 如果是Clone下來的solution，請先還原nuGet套件(否則Bot Framework SDK無法使用) <br>
  Tools → Nuget Package Manager → Nuget Package Console → 輸入"dotnet build"，即開始還原套件檔案  <br>