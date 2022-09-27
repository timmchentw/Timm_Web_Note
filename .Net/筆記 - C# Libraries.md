# C# Libraries

## Tools

* ### Newtonsoft.Json

    Json轉換神套件

* ### Serilog
  
    Logger整合，有多種Sink管道可擴充

* ### Moq

    Unit Test製作假物件

* ### Automapper

    快速Mapping兩個相似物件，精簡Script (小心Property Reference會比較難找)

* ### AspectInjector

    AOP套件，搭配Attribute使用

* ### FluentAssertions

    加強版Test Assert，更口語化表達、且可以互相比較Object values equivalence

    ```C#
    // 簡單比較
    result.Should().Be(expect);
    // 物件比較 (可揭露哪些值有問題)
    result.Should().BeEquivalentTo(expect);
    // 物件比較 (可設定跳過某些欄位)
    result.Should().BeEquivalentTo(expect, (option) => option.Excluding(x => x.Column1));
    ```

## Files

* ### NPOI

    讀取Excel檔案

    ```C#
    待補充Generic Read Excel方法
    ```

    * [[C#]使用NPOI產生Excel](https://dotblogs.com.tw/mileslin/2016/02/05/101527)
    * [How to set Validation for a cell in Excel created using NPOI](https://stackoverflow.com/questions/17210373/how-to-set-validation-for-a-cell-in-excel-created-using-npoi)

    #### NPOI.Mapper

    自動Mapping Excel Model的補充套件
    * [通過 Npoi.Mapper + 強型別 Model 讀寫 Excel](https://dotblogs.com.tw/yc421206/2020/08/16/via_npoi_mapper_strong_type_model_write_read_excel)

* ### CsvHelper

    讀取CSV檔案

    ```C#
    待補充Generic Read CSV方法
    ```

## Web

* ### Htmlagilitypack

    讀取HTML為DOM物件

* ### Selenium

    網路爬蟲工具，須搭配瀏覽器

* ### NSwag.AspNetCore

    Swagger API文件工具

## ORM

* ### Dapper

    輕量化SQL套件，簡化ADO.Net流程

* ### Microsoft.EntityFrameworkCore

    官方Database套件，有多種模式

## OWIN

* ### Hangfire

    套件管理架構，內建Dashboard

## Cloud

* ### Azure.Storage.Blob

    連接Azure網路儲存體

* ### Microsoft.ApplicationInsights

    連接Azure Application Insights紀錄App執行歷程與LOG

* ### 