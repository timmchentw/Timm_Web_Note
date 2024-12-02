# .net MVC

## Action

* 注意Controller & Action Route的部分，最好預先設定好 (比如`[Route("...")]`)

### GET (讀入網頁、回傳預設數值)
```C#
using MyProj.Models.viewmodel;  // 引入viewmodel目錄

[httpGet]
public ActionResult MyAction(MyViewModel model)
{
    MyData = "Here's the default data";
    model.data = MyData;    // 輸入預設值進View Model
    return(model);
}
```

其中View Model內容為
```C#
namespace MyProj.Models.viewmodel
{
    public class MyViewModel
    {
        public string data { get; set; }
    }
}
```


### POST (接收前端請求&附加資訊、回傳Json結果)
#### <接收參數>
```C#
[httpPost]
public JsonResult MyAction(string myVar)
{
    var myData = myVar + " has been processed."
    return Json(myData)
}
```

其中前端給予的Json資訊必須與後端指定變數同名 (即myVar)
```javascript
data: JSON.stringify({ myVar: 'The content' })    // fetch的data改為body
```

### POST (接收前端請求&附加資料於view model並回傳Json結果)
#### <接收View model>
```C#
[httpPost]
public ActionResult MyAction(myViewModel model)
{
    var myData =  model.myVar
    return Json(myData)
}
```

其中前端必須先匯入viewmodel數值
```javascript
data: JSON.stringify({ Model: model })
```

## TempData

* 用於跨Controller Action & View的資料傳遞


## LINQ ENumerable

### 資料處理指令
* Concat 合併

```C#
A.Concat(B) // A與B之Row合併
```

* Select

```C#
A.Select(x => x.num) // 選定num行、或改名及前處理 (ex. ToString())，相當於SQL的select ...
```

* SelectMany

```C#
A.SelectMany(x => x.listProperty) // 將List的某Property攤平成一個List，相當於SQL的union
```

* Take

```C#
A.Take(50) // 只取TOP 50個資料列
```

* Contains

```C#
A.Contains(B) // A中任何含有B之部分
```

* Where

```C#
A.Where(x => x.num == '1' ) // 篩選A中num為1者
```

* Except

```C#
A.Except(AExcept, new AComparer()) // A中排除掉AExcept的資料(A與AExcept需為同個class object); AComparer負責比對某個行
```

* Any
* Skip
* Order

### LINQ
```C#
from a in DBTable1
join b in DBTable2 on a.id equals b.id
join c in DBTable3 on a.id equals c.id into ps from c in ps.DefaultIfEmpty()    // Left Join
where a < 10 || b < 10
select a
// New { ... } 可選取特定欄位、或甚至改名或前處理
// Group也可用
```

### Entity Framework
#### 建立DataBase First EF
1. 在指定資料夾Add → New Item... → ADO.Net Entity Data Model
2. EF Designer from Database

[參考教學](https://www.c-sharpcorner.com/article/create-and-update-an-edmx-file-using-entity-framework-data-model-in-visual-stud/) <br>

#### DbContext
* 新建entity連線
```C#
new Db_Entity.MyDB1();
new DbContext("MyDB1");
```
* 完整entity連線流程
```C#
// New entity (Table schema)
var myEntity = new Db_Entity.MyDBTable1() 
{
    COL1 = "123",
    COL2 = 0
}
```
```C#
// Connect to DB
DbContext myDbContext = new Db_Entity.MyDB1();
myDbContext.Add(myEntity);
myDbContext.SaveChanges();
SaveChanges.Dispose()

// or below
using (DbContext myDbContext = new Db_Entity.MyDB1())   // 可省略dispose的部分
{
    myDbContext.Add(myEntity);
    myDbContext.SaveChanges();
}
```

Q: 在呼叫 DbContext 的 SaveChanges 方法之後，在SSIS設定default value的欄位值卻仍是 null。 <br>
A: 在 Visual Studio 中開啟該.edmx檔案，點選含有預設值的該欄位，然後到properties中，將 StoreGeneratedPattern 屬性值由預設的 None 改為 Computed

### 自訂Icomparer
```C#
public class MyComparer : IEqualityComparer<MyT>    // 實作IEqualityComparer功能
{
    public bool Equals(MyT x, MyT y)                // Equals為內建必要功能，用於比較x與y兩者之某參數
    {
        return x.name == y.name;                    // 指定比較參數(重要)
    }
    public int GetHashCode(MyT model)               // GetHashCode為內建必要功能，抽取比較參數
    {
        return model.name.GetHashCode();            // 指定參數(重要)
    }
}

public class MyT                                    // 比較用的物件範例
{
    public string name;
    public string id;
    public string email;
}
```
* Except
```C#
// myExceptDT為例外名單的DataTable，目標行名稱為"COL_FROM_DATATABLE"
var myExcept = myExceptDT.Select(x => new MyT() { name = x.Field<string>("COL_FROM_DATATABLE") }).ToList(); // 例外名單DT -> MyT格式
var myExceptResult = myList.AsEnumerable().Except(myExcept, new MyComparer())   // 比較兩者皆使用MyT格式，就可以使用Except進行比較排除
```

## Unit Test


## Razor

### 前端隱藏賦值
```razor
<script>
    console.log( $("#myHtml").val() )    // 從html取值
</script>

@Html.Hidden("myHtml", (bool)ViewBag.myHtml)                @* 將ViewBag賦值到html並隱藏 *@
@Html.HiddenFor(model => model.myId, new { id = "myId" })   @* 將model.myId賦值到html並隱藏 *@

<Html>
...
</Html>

```

## Web.config

.net MVC
設定可跳過My AuthorizeAttribute (不須驗證登入即可進入網頁)

```xml
<configuration>
  <location path="MyController/MyAction">
    <system.web>
      <authorization>
        <allow users="?" />
      </authorization>
    </system.web>
  </location>
</configuration>
```

## Authentication

## Model Validation 後端驗證

### Attributes

* `Required`
* `EmailAddress`
* `RegularExpression("...". ErrorMessage = "...")`

### System.Web.Mvc.ModelState

* ModelState.IsValid
* ModelState.Select(x => x.Value.Errors)

```C#
var validateErrors = ModelState.Select(x => x.Value.Errors).Where(y => y.Count > 0).ToList();
string errorMsg = String.Join("; ", validateErrors.Select(x =>
{
    string validateMsg = x.FirstOrDefault().ErrorMessage;
    if (String.IsNullOrEmpty(validateMsg))
    {
        validateMsg = x.FirstOrDefault().Exception.Message;
    }
    return validateMsg;
}));
return errorMsg;
```

## Dependency Injection

### Microsoft.Extensions.DependencyInjection

如同.net core內建的DI，官方套件同樣支援.Net 4.6.1以上版本

```C#
```


* 參考來源: [余小章-如何使用 DI Container for Microsoft.Extensions.DependencyInjection](https://www.dotblogs.com.tw/yc421206/2020/10/29/standard_di_container_for_microsoft_extensions_dependencyInjection)

