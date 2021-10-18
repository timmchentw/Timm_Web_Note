[JavaScript](#JavaScript) </br>
[JQuery](#JQuery) </br>
[ReactJS](#ReactJS) </br>
[AngularJS](#AngularJS) </br>
[Vue](#Vue) </br>

# JavaScript
* 載入與初始化 (載入外部套件可使用網址)
```html
<script type="text/javascript" src="~/Scripts/JS套件.js"></script>
```

* => (簡化function) </br>
**單行簡化** (無x則為空括號)
```javascript
var func = x => x+1
// 原型為
var func = function(x) return(x+1)
```
**多行簡化**
```javascript
var func = x => {
  var calc = x+1; 
  return calc
  }
// 原型為
var func = function(x) {
  var calc = x+1; 
  return calc
}
```

* HTML DOM控制

| document | 說明 |
| --- | --- |
| `.getElementByName()` | 取得DOM |
| `.getElementsById()` | 取得DOMs (Array) |
| `.getElementById().value` | 取得值 |
| `.CreateElement()` | 創造DOM |
| `.evaluate(xpath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue` | 以Xpath搜尋第一個DOM節點 |
| `.addEventListener("input", function() {});`| 當input改變時執行 |
| －－－網址控制－－－ | |
| `.location.href` | 完整URL |
| `.location.host` | 域名+端口號 |
| `.location.hostname` | 域名 |
| `.location.hash` | #後部分 |
| `.location.search` | ?後部分 |
| `.location.reload()` | 刷新/開新網頁 |
| `.location.replace(URL)` | 開新網頁 |

* 下載功能
```javascript
var a = document.createElement('a');    // Create hyperlink for download
var url = "下載網址在這裡";
a.href = url;
a.download = url;
a.click();
```

## 資料處理
### Object
|語法|說明|
|--|--|
| `Object.assign({}, ori, new)`     | 不覆蓋情況下繼承(與覆蓋同key值)到原本的object，產生新的object |
| `Object.values(obj)`              | 獲得obj內值組成的Array  |
| `Object.keys(obj)`                | 獲得obj每個key值組成的Array  |
| `Object.keys(obj).length === 0 && obj.constructor === Object)`| 確認obj為空的Object |
### Array
|語法|說明|
|--|--|
| `Array.from(obj)`                 | 轉換obj類陣列資料並產生新的Array |
| `myArray.map((item, [index])=>{ })` | 迭代myArray每列並回傳結果為Array，包含各列元素(item)與編號(index) |
| `myArray.concat(myArray2)`        | 列合併兩個Array |
| `myArray.find(x => x === value)`  | 查詢Array中元素 |
| `myArray.slice([begin], [end])` | 取得begin到end之間的Array |
| `myArray.splice(begin, [deleteNum], [newArrayItem])` | (會改變原本myArray)取代begin後的array items，替換為新的，並回傳"被切掉"的部分 |
| `myArray.split(seperator, [num])` | 以符號去切割字串 |

## ES6
* let   可變變數，且只存在該循環中(local)，不會更改原變數記憶體 <br>
* const 不可變變數，不會更改原變數記憶體


# JQuery
* 第一次使用須於html引用
```html
<script src="~/Scripts/jquery-3.3.1.min.js"></script>
```
* 網頁載入完成時直接執行
```javascript
$(document).ready(function(){
  // TYPE SCRIPT HERE
});
```
或簡寫為：
```javascript
$(function(){ 
  // TYPE SCRIPT HERE 
});
```
* $(this)
$('...').on(...)的事件中，<br>
可使用$(this)指到目前的DOM

### 基本功能

| $("#id") | 說明 |
| --- | --- |
| `.change()` | 值改變時觸發 |
| `.val()` `.val("")` | 取得值/改變值 |
| `.click()` | 直接點擊/被點擊時 |
| `.select()` | 直接點擊/被點擊時 |
| `.empty()` | 清空子節點 |
| `.remove()` | 刪除該節點 |
| `.before()` `.after()` | 插入HTML項目到該節點前/後 |
| `.prepend()` `.append()` | 插入HTML子項目該節點最前/後 |
| `.contents` | 填入內容 |
| `.find` | 尋找 |
| `.show` | 顯示 |
| `.hide` | 隱藏 |
| `.children()` | 子元素 |
| `.parent()` | 父元素 |
| `.first()` | 第一個 |
| `.siblings()` | 每個都找 |
| `.each(function() { ... })` | 每個都... |
| `.eq()` | 第幾個(index) |
| `.closest()` | 找最近的 |
| `.hasClass('...')` | 判斷是否含有class name |
| `.trigger()`| 觸發事件 |
| `.prop('prop_name', new_value)`| 更改property內容 |

觸發動作後，須在括號內加入`function() {...}`

### 套件與功能
* [Dialog](https://jqueryui.com/dialog/) 彈出視窗
* [DataTables](https://datatables.net/) 動態表格
* [Dropzone](https://www.dropzonejs.com/) 拖拉式上傳
* [Collapse (bootstrap)](https://www.runoob.com/bootstrap/bootstrap-collapse-plugin.html) 開闔式按鈕
* [zTree](http://www.treejs.cn/v3/main.php#_zTreeInfo) 樹狀圖選單
* [tokenInput](https://loopj.com/jquery-tokeninput/) 搜尋建議標籤 
* [redactor](https://imperavi.com/redactor/) HTML編輯器
* 使用`<input type="file" id="file">`在前端取得檔案
```javascript
$('#file').change( function(e) {
  var uploadedFiles = e.target.files
  var filename1 = uploadedFiles[0].name
});
```

# [ReactJS](https://github.com/timmchentw/Web-Note-Text/blob/master/%E7%AD%86%E8%A8%98%20-%20JS%20React.md)
# AngularJS
# Vue
