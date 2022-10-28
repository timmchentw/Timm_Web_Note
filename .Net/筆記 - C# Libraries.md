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

    讀寫Excel檔案 </br>
    ※Create/Read Column & Row的作法較為傳統，可改用Npoi.Mapper進行強型別轉換

    ```C#
    using NPOI.XSSF.UserModel;

    public byte[] WriteExcel(IWorkbook workbook)
    {
        var stream = new MemoryStream();
        workbook.Write(stream);

        return stream.ToArray();
    }

    public IWorkbook ReadExcel(IFormFile file)
    {
        XSSFWorkbook workbook = new();
        using (var ms = new MemoryStream())
        {
            file.CopyTo(ms);
            Stream stream = new MemoryStream(ms.ToArray());

            workbook = new XSSFWorkbook(stream);
        }

        return workbook;
    }

    public IList<T> ReadRowData<MyClass>(IWorkbook workbook)
    {
        var results = new List<MyClass>();

        XSSFSheet u_sheet = (XSSFSheet)workbook.GetSheetAt(0);  // first sheet
        XSSFRow firstRow = (XSSFRow)u_sheet.GetRow(u_sheet.FirstRowNum);

        for (int i = (u_sheet.FirstRowNum + 1); i <= u_sheet.LastRowNum; i++)
        {
            XSSFRow row = (XSSFRow)u_sheet.GetRow(i);  // do not get header row
            var result = new MyClass();

            result.Name = this.GetCellValue(row, 0);
            result.Date = Convert.ToDateTime(this.GetCellValue(row, 1));
            results.Add(instance);
        }

        return results;
    }

    // 以下方法可避免"公式"欄位取得到公式本身，而可以直接取得"Value"
    private string? GetCellValue(IRow row, int colNo)
    {
        var cell = row.GetCell(colNo);

        if (cell == null)
        {
            return null;
        }
        switch (cell.CellType)
        {
            case CellType.Error:
                throw new NotImplementedException("Error exist in sheet file");

            case CellType.Formula:
                switch (cell.CachedFormulaResultType)
                {
                    case CellType.String:
                        return cell.StringCellValue;
                    case CellType.Numeric:
                        return cell.NumericCellValue.ToString();
                    case CellType.Boolean:
                        return cell.BooleanCellValue.ToString();
                    default:
                        return cell.ToString();
                }

            default:
                return cell.ToString();
        }
    }
    ```

    * [[C#]使用NPOI產生Excel](https://dotblogs.com.tw/mileslin/2016/02/05/101527)
    * [How to set Validation for a cell in Excel created using NPOI](https://stackoverflow.com/questions/17210373/how-to-set-validation-for-a-cell-in-excel-created-using-npoi)

    #### Data Validation

    可設定下拉選單、欄位驗證等功能

    ```C#
    using NPOI.SS.UserModel;
    using NPOI.SS.Util;
    using NPOI.XSSF.UserModel;

    public void SetDataValidation(ISheet sheet)
    {
        var helper = new XSSFDataValidationHelper((XSSFSheet)sheet);
        var dvConstraint = new XSSFDataValidationConstraint(new string[] {"Option1, Option2"});
        var markColumnSet = new CellRangeAddressList(firstRow: 1,
                                                    lastRow: 1000,
                                                    firstCol: 3,
                                                    lastCol: 3);
        var dataValidationSet = helper.CreateValidation(dvConstraint, markColumnSet);

        dataValidationSet.EmptyCellAllowed = true;
        dataValidationSet.CreateErrorBox($"Wrong Value of column 3. Please Enter a correct value");
        sheet.AddValidationData(dataValidationSet);
    }
    ```

    #### NPOI.Mapper

    自動Mapping Excel Model的補充套件

    ```C#
    using NPOI.SS.UserModel;
    using Npoi.Mapper;

    public IEnumerable<MyClass> ReadWorkbook(IWorkbook workbook)
    {
        var mapper = new Mapper(workbook);
        mapper.Map<MyClass>("Name (Excel Column)", x => x.Name));
        mapper.Map<MyClass>("Date (Excel Column)", nameof(MyClass.Date));

        var result = mapper.Take<MyClass>(0);

        return result.Select(x => x.Value);
    }

    public IWorkbook WriteWorkbook(IEnumerable<MyClass> data)
    {
        var mapper = new Mapper();

        // Mapping Excel欄位&Property (兩種皆可)
        mapper.Map<MyClass>("Name (Excel Column)", x => x.Name));
        mapper.Map<MyClass>("Date (Excel Column)", nameof(MyClass.Date));
        // 設定格式
        mapper.Format<MyClass>("M/D/YYYY", x => x.Date)
        // 置入資料
        mapper.Put<MyClass>(data, sheetIndex: 0, overwrite: true);

        // 儲存檔案
        mapper.Save("C:/MyPath");

        // 也可直接取得NPOI Workbook實體，功能互補
        return mapper.Workbook;
    }
    ```

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