# C# 筆記
* [字串處理](#字串處理)
* [變數型態](#變數型態)
* [存取授權](#存取授權)
* [建構子](#建構子)
* [存取授權](#存取授權) 
* [靜態修飾](#靜態修飾) 
* [繼承性](#繼承性) 
* [運算子](#運算子)
* [判斷式與迴圈](#判斷式與迴圈)
* [錯誤處理](#錯誤處理)

[用Dict去做掃描比LINQ Where速度快上10倍以上](https://blog.darkthread.net/blog/linq-search-performance-issue)

## **《字串處理》**
| 功能 | 語法 |
|-----|-----|
| 黏合字串與變數  |  `$"ABC....{Var or "string"}...."` |
| 黏合字串與變數2 | `$"...{0}...{1}...{2}...", var1, var2, var3` |
| 黏合字串與變數3 | `"String" + Var` |
| 字串中的""    |  `@"....\"......\"..."` |
| 字串中的{}    |  `@"....{{......}}..."` |
| 字串中的IF    |  `$"....{(Var==1 ? "string1": "string2")}"` |
| 字串長度      |  `{Var.Length} -> "3"` |
| 取代          |  `.replace(OldChar. newChar)` |
| 去除頭尾      |  `.trim()`    |
| 去除尾字      |  `.trimEnd(myChar)` |
| 去除前字      |  `.trimStart(myChar)` |
| 去除特定區間  |  `.remove(ref_pos, char_num)` |
| 分割         |  `.split(myChars)` |
| 檢查         |  `.contains("something")` |
| 轉換大小寫    |  `.ToUpper`  `.ToLower`  `.TitleCase`(字首大寫) |
| 轉為數字      |  `int.Parse("String")` |
| 檢查是否為空  |  `string.isNullorEmpty("String")` |

### 檔案字串
| 功能 | 語法 |
|-----|-----|
| 取得檔名(不包含附檔名)  |  `Path.GetFileNameWithoutExtension(filePath)` |
| 取得副檔名             | `Path.GetExtension(filePath)` |
| 資料根目錄             | `Path.GetPathRoot(filePath)` |
| 取得路徑               | `Path.GetFullPath(filePath)` |


### Regex (Regular Expression字串格式檢查)
| 功能 | 語法 |
|-----|-----|
| 允許範圍 | `[a-z]` `[0-9]` |
| 字元集合 |  `(...)` |
| |
| 首字為a    |    `^a` |
| 尾字為a    |    `a$` |
| 前面有a可出現1次以上 | `a+` |
| 前面有a可出現1次以下 | `a?` |
| 前面有a可出現或不出現 | `a*` |
| 旁邊出現A     |  `A\b` |
| 旁邊不出現A    | `A\B` |
| 任何字元(\n外) | `.` |
| 任何字元(含\n) | `[\s\S]` |
| 任何英文字元   | `\w`  |
| 任何非英文字元 | `\W` |
| 任何空白字元   | `\s` |
| 任何非空白字元 | `\S` |
| 任何數字字元   | `\d` (包含"一二...") |
| 任何非數字字元 | `\D` |
| |
| ab以外所有   |  `[^ab]` |
| abc擇一即可  | `[abc]` |
| abcd中排除bc  | `[[a-d]-[bc]]` |
| a或b或c       | `(a\|b\|c)` |
| |
| a重複n次    |  `a{n}` |
| a重複n到m次  | `a{n,m}` |
| a至少重複1次 | `a{n,}` |
| |
| 所有大寫字母 |   `(?-i:\p{Lu})` |
| 所有非大寫字母 | `(?-i:\P{Lu})` |
| 檢查中文字母 |   `\p{IsCJKUnifiedIdeographs}` |

| **特殊符號** | 語法 |
|-----|-----|
| (     |     `\(` |
| /     |    `//` |
| .     |     `\.` |
| +     |     `\+` |
| ?     |     `\?` |
| *     |     `\*` |
| 後退鍵 |     `[\b]` |
| 換行   |     `\r\n` |
| TAB    |    `\t` |

| 表達 | 語法 |
|-----|-----|
| 表示式 |    `var regex = /expression/` |
| 進行檢查 |  `regex.test("string")` |


## 變數型態

### 基本型態
* 數字邏輯: `int` `float` `double` `decimal`(指定位數) `bool`(True & False)
* 其他: `string` `char` `object` `interface` `delegate`

### 資料結構
| 型態 | 名稱 | 特性 | 範例|
|-----|-----|-----|-----|
| Array | 陣列 | 須宣告型別+固定資料長度 | `int[] newArray = new int[3]{777,666,123};` |
| List | 列表 | 須宣告型別+可插入(.Insert)移除(.Remove) | `List<string> newList = new List<string>();`  <br>  `newList.Add("newString");` |
| Datatable | 資料表 | 複合資料型態+分別插入Col與Row | `DataTable dt = new DataTable();`  <br>  `dt.Columns.Add("cName", typeof("string"));`  <br>   `Datarow row = dt.NewRow();`  <br>  `row["cName"] = ...`  <br>  `dt.Rows.Add(row)`|
| Stack | 堆疊 |  |  |
| Queue | 佇列 |  |  |
| Dictionary | 字典 |  |  |
| Tuple | 雙參數 |||

## 存取授權
| 存取類別 | 說明 |
| ----- | ----- |
| public | 可公開存取 |
| private | 同class可存取 (不可被引用或呼叫)| 
| protected | 同class 或 原class衍伸之class可存取 | 
| internal | 同project可存取 |  

void 物件無輸出 (不需return)
static 靜態修飾 (可直接呼叫不需new、可防止改動、同class可供全部共用)  <br>

## 建構子
Contructor  <br>
1. 無output，可input (有即強迫使用者輸入input)
2. 與某class同名，且無型別
3. 可直接執行與賦值
4. input可overloaded (創很多個constructor，每個input都不一樣)
```C#

```

## 靜態修飾
Static  <br>
**(1) 可直接呼叫**
```C#
class Math
{
  public *static* int max(int a, int b)
  {
    int result;
    if (a < b) result = b;
    else result = a;
    return result;
  }
}
```
使用時不必new新物件(Math m = new Math)  <br>
```C#
int a = 10;
int b = 20;
int c = Math.max(a,b)
```

**(2) 同class可供全部共用**  <br>
於class Student中 (如[建構子](#建構子)所示)  <br>
```C#
// 加入靜態宣告
public static int PassGrade = 2;
```
```C#
// 當使用此class時，更改靜態宣告值
Student.PassGrade = 1; 
```
```C#
// 更改結果可供全class使用
s1.isPass() -> True
```
p.s.
```C#
在Class Student中新增:
pulic bool isPass()
{
  if (Grade >= PassGrade) return true;
  else return false
}
```


## 繼承性
```C#
class B : A         //B繼承A之屬性(內建變數、Function等)
{...}
```

## 修飾詞
| Base基底修飾 | 說明 |
| -- | -- |
| `abstract` | 未具現化，需實作才可使用 (not instantiated) |
| `virtual` | 可直接具現化，並可以被修改覆蓋 |

| Derived衍伸修飾 | 說明 |
| -- | -- |
| `override` | 覆蓋原有base class的功能 (`BaseClass bc = new DeriveClass()`也同樣適用) |
| `new` | 不覆蓋原有base class的功能，但可以做擴展 |

`sealed` 防止被繼承衍伸 <br>

[微軟範例](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/knowing-when-to-use-override-and-new-keywords) <br>


## 運算子
| 運算子 | 意義 |
| --- | --- |
| `++` `--` | ±1 | 
| `&&` `\|\|` | AND / OR  |
| `==` `!=` | 等於 / 不等於  | 
| `+=` `-=` | 原變數再加/減多少 |
| `/` `%` | 除取商數/餘數 |

## 判斷式與迴圈
### **IF** 
`if (!var)` → `if(int==0)` or `if(string==null)`  <br>
!代表相反之涵義

### **?: ??**
?: 簡化條件式為單行
```C#
int a = b > 0 ? b : c;
// 宣告a整數值，當b>0時則a=b，否則為a=c
```
?? 簡化檢查null值
```C#
int a = b ?? c ?? d;
// 宣告a整數值，當b為非null時a=b；當b為null時a=c；當c為null時a=d...
```

### **SWITCH** 
簡化版IF
```C#
switch (var)
{
  case 1: ...
    break
  case 2: ...
    break
  default : ...
    break
}
```

### **FOR**
```C#
for (int i=0; i<10; i++)
{
  i ;
}
```

### **FOREACH** 
將LIST逐層處理
```C#
foreach (var item in a_LIST)
{
  item ;
}
```

### **TRY & CATCH** 
嘗試執行與失敗接住
```C#
try
{
  try something here
}
catch (Exception e)
{
  go to here if error occured
  console.log(e)   // Explain the error
}
finally 
{
  any condition will go to here
}
```

### HTML處理
#### HtmlAgilityPack
* HtmlDocument
```C#
HtmlDocument doc = new HtmlDocument();
doc.Load('File Path String')
// doc.LoadHtml('Html String')
```
* HtmlNode
```C#
HtmlNode node1 = doc.CreateTextNode('Html Node String');
doc.DocumentNode.AppendChild(node1);  // 對整個HTML插入子節點 (PreoendChild / AppendChild)

HtmlNode node2 = doc.SelectSingleNode('XPATH');
node2.ParentNode.InsertBefore(node1);  // 選定節點並插入子節點 (InsertBefore / InsertAfter)
```
* 取出Html (String)
`doc.DocumentNode.InnerHtml`

### 檔案讀寫
* StreamReader / StreamWriter  `讀寫文字檔`
```C#
StreamReader sr = new StreamReader(filepath, Encoding.UTF8);   // Read HTML file
            string myString = sr.ReadToEnd();
            sr.Close();
            sr.Dispose();
// 寫入則為：sw.Write(myString)，其他宣告與指令相同
```
* 

## 注意事項
* 使用WebClient載入https連結時，如.net framework版本較舊、憑證版本較新，須先預設較新版本的SSL/TLS憑證
```C#
System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;  // 定義TLS1.2版本

WebClient wc = new WebClient();
MemoryStream ms = new MemoryStream(wc.DownloadData(URL));
HtmlDocument doc = new HtmlDocument();
doc.Load(ms, Encoding.UTF8);
```

## Cache快取

* MemoryCache (存在全域，不隨Http Request而變)
* ObjectCache 
* HttpContext Cache (隨著Http Request而變，可為各個user個別存取)

```C#
List<string> currentCache = HttpContext.Current.Cache["cacheName"] as List<string>
List<string> newCache = new List<string>();

if (!Enumerable.SequenceEqual(currentCache, newCache)) // 自訂時機判斷更新Cache (此例子比較兩個List值是否相同)
{
    // Insert -> 覆寫 Override current cache ;  Add -> 不覆寫 Add cache only when currentCache is null
    HttpContext.Current.Cache.Insert("cacheName",
                                     newCache,
                                     null,
                                     DateTime.Now.AddDays(1),                       // absoluteExpiration -> 指定時間刪除 (通常設定DateTime.Now往後推多久)
                                     System.Web.Caching.Cache.NoSlidingExpiration,  // slidingExpiration ->  最後一次取用後多久刪除 (有存取就會重新計算)
                                     System.Web.Caching.CacheItemPriority.High,     // 設定快取優先度
                                     new System.Web.Caching.CacheItemRemovedCallback(RemovedCallback)); // 刪除快取後的callback行為 (通知程式快取已刪除用)
}
```

```C#
public static void RemovedCallback(string k, object v, System.Web.Caching.CacheItemRemovedReason r)
{
    itemRemoved = true;
    reason = r;
}
```

## 錯誤處理
* [.Net MVC 六種處理exception的方式(包含controller接錯)](http://programming.signage-cloud.org/dotnet_mvc_exception_handle/)
* [請直接Throw而非Throw ex防止資訊丟失](https://www.dotblogs.com.tw/wasichris/2015/06/07/151505)
* [包裝程式中共用的 try...catch(簡化try catch)](https://dotblogs.azurewebsites.net/rainmaker/2014/11/19/147361)
* [建立客製化exception必要步驟](https://rainmakerho.github.io/2018/08/21/2018032/)
* [StackOverFlow正確建立可序列化Exception](https://stackoverflow.com/questions/94488/what-is-the-correct-way-to-make-a-custom-net-exception-serializable)

## Recursion (遞迴)
```C#
public static void RecursionMethod(string input)
{
    RecursionMethod(input);
    //...
}
```

## Delegate

```C#
public static void TryToDo(Func<string, bool> delegateMethod)
{
    string something = "Test";
    try
    {
        bool result = delegateMethod(something);
    }
    catch
    {
        // Log
    }
}
```

## Expression

非Runtime的表示式 (適合用於欄位Mapping、延遲執行等)，常見例子為LINQ

```C#
Expression<Func<string, int>> example = (stringInput => Convert.ToInt32(stringInput));

Expression<Func<MyClass, object>> example2 = (x => x.Prop1);
```

## Yield

分批作業

```C#
public IEnumerable<string> Sample() 
{
    for (i = 0; i < 10; i++)
    {
        yield return i;
    }
}
```

## Class
### Abstract Class & Interface
### Partial Class

## 網址與路徑

### Path

  ```C#
    Path.Combine("directory/root/", "/relativePath/file");
    Path.GetFileName("fullpath");
    Path.GetExtension("fullpath");
  ```

## Stream

```C#
var fileStream = new MemoryStream();
await file.CopyToAsync(fileStream);
fileStream.Position = 0;    // 重置讀取位置，方便下一次Stream被讀取時從頭開始 (否則會有Exception)
```

## Expression-bodied 
#### Expression-bodied Methods
    MyFunction(int a, int b) =>a + b;
#### Expression-bodied members

## Asynchronize
#### async/await

* `GetAwaiter`
* `Task.CompletedTask`
* `XXXAsync().ConfigureAwait(false)` <br>
    Why? 參考[About ConfigureAwait](https://medium.com/ricos-note/about-configureawait-5f173cd5f4f)

## Reflection

### typeof

### attributes

### CreateInstance

```C#
Type type = Type.GetType(name, true);
object instance = Activator.CreateInstance(type);
PropertyInfo prop = type.GetProperty(property);
prop.SetValue(instance, value, null);
```

Reference:
* [小山的教學平台](https://www.youtube.com/channel/UCmumrs_hb9s6eoVI29gLBgA) (建構子、靜態修飾、繼承性)
