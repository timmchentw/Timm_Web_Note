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


## 架構筆記
1. 如果不使用DI相依性注入，有在Constructer New出來使用的class，在結束使用後記憶體將暫時不會釋出，尤其連線相關的variable，因此最好手動dispose
ps. Controller呼叫new service，new service時也會new DB connection，如果request完service並沒有dispose connection就會出現問題 (unit of work會出現、但不使用unit of work則應該每次呼叫DAL時都會using connection -> using完會自動dispose) <br>
2. Unit of work為集中connection的方法，將各自獨立的repository共用SqlConnection，並使用SqlTransaction將各次的SQL指令作為一個單位，如果該單位沒有成功Save change，則所有的改動都可以還原到最開始的狀態 (rollback) <br>
3. 程式架構 View(前端) -> Controller(資料轉換、方法呼叫) -> Service Layer(邏輯處理、呼叫資料源) -> Data Access Layer(控制資料庫、API等連線)，各層間不可以反向引用(但可反向依賴interface)，達成封裝性的要求，以避免耦合過多，如要跨層使用方法可用static方法寫成Utility共用 (如Enum、Util方法等) <br>
4. 如物件導向有類似功能但又不盡相同的class，可將共有的變數、方法提升為base class，再利用繼承擴充base class，擴充方法如果都差不多則用Abstract, Interface去規定名稱、型別等，以避免類似的class中存在大量重複的code、類似方法規格不統一等
