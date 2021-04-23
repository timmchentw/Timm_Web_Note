# Text Template Transformation Toolkit (T4)
## 用於自動生成程式碼

[基本介紹](https://docs.microsoft.com/zh-tw/visualstudio/modeling/design-time-code-generation-by-using-t4-text-templates?view=vs-2019) <br>
[專用指示詞](https://docs.microsoft.com/zh-tw/visualstudio/modeling/t4-template-directive?view=vs-2019) <br>
[控制區塊](https://docs.microsoft.com/en-us/visualstudio/modeling/text-template-control-blocks?view=vs-2019) <br>

### 語法說明
| Control block語法 | 名稱 | 用途 |
| -- | -- | -- |
| <#@ ... #> | Template Directive | 範本指示詞，用於設定輸出與程式等 |
| <# ... #> | Standard control blocks | 程式主體 |
| <#+ ... #> | Class feature control blocks | 用於放置擴充類別、函式等，可供程式主體使用、共用 |
| <#= ... #> | Expression control blocks | 供程式輸出成文字，相當於Console.log(...) |
| ... | No control blocks | 直接輸出文字 |

### 語法特性
因撰寫的內容分成"程式主體"與"輸出文字" (用control blocks隔開)，
所以內容會較為難理解(各自都有排版)，
如果將程式(C#)與文字分開來看會較好維護，
另外以下為各部分注意事項：
* **程式部分** - 不會因為文字block而被隔開，因此整個檔案可視為一個整體 (把文字部分當Console.log即可)
* **文字部分** - 打出來的內容即為輸出結果，因此要注意排版
* **小撇步** - 先用程式block把要的資料取得後，把文字部分當主體，再把輸出用的程式包起來取得變數(<#= ... #>)


### 撰寫範例
| 程式碼 | 補充 |
| -- | -- |
| `<#@ template debug="true" hostspecific="true" language="C#" #> ` | 設定config |
| `<#@ import namespace="System.Data" #>`  | 相當於C#的using ...; |
| `<#@ assembly name="System.ComponentModel" #>`  | 加入reference |
| `<#@ output extension="cs" #>` |  輸出成C# class 格式 |
| `<# string myText = "Hello World!" #>` | 主程式，不輸出不會出現在結果當中 |
| `<#= myText #>` |  會輸出為" myText " |
| `<#+ public static string MyFunction(string myText) { ... } #>` | 主程式可直接呼叫 |

### 後續更新
如資料是從DB撈出，
如DB有資料更新，
將tt檔案右鍵Run Custom Tool就可以更新為新的資料
