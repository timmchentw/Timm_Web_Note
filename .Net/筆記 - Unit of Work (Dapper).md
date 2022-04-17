## Unit of work
(1) Unit of work基本包含Begin、Save & Reset (Transaction)，用於批次交易寫入<br>
(2) Connection與Transaction將傳入Repository作為共用<br>
(3) Unit of work以Database為單位，其下引入需使用到的Repositories
```
public interface IUnitOfWork : IDisposable
{
    IDbConnection Connection { get; }
    IDbTransaction Transaction{ get; }
    IDapperRepository GenericRepo { get; }

    void Begin();
    void Save();
    void Reset();
}
```

## Dapper Repository
(1) Get & Execute方法實作Dapper方法<br>
(2) Connection & Transaction則在Constructor時就須傳入做共用
```
public interface IDapperRepository
{
    List<T> Get<T>(string sql, object parameters = null, int? timeoutSeconds = null);
    int Execute(string sql, object parameters = null, int? timeoutSeconds = null);
}
```

## T4 Dynamic Data Model
(1) 利用SQL: INFORMATION_SCHEMA.COLUMNS，取得Table Columns Names & Type
```
WITH TABLE_COLUMN_COUNT AS (
    SELECT TABLE_NAME, MAX(ORDINAL_POSITION) AS MAX_COLUMN_NUM
    FROM INFORMATION_SCHEMA.COLUMNS
    GROUP BY TABLE_NAME
)
SELECT a.TABLE_NAME, a.COLUMN_NAME, a.ORDINAL_POSITION, a.IS_NULLABLE, a.DATA_TYPE, b.MAX_COLUMN_NUM, 
        ISNULL(REPLACE(REPLACE(c.DEFAULT_VALUE, '(', ''), ')', ''), 'NULL') AS DEFAULT_VALUE
FROM INFORMATION_SCHEMA.COLUMNS a
INNER JOIN TABLE_COLUMN_COUNT b on a.TABLE_NAME = b.TABLE_NAME
WHERE a.TABLE_NAME in (
    'Profile_Portfolio'
    )
```
(2) 利用Data type自行Mapping為C#型別，以建立C# Classes
```
public static string GetNetDataType(string sqlDataTypeName)
{
    switch (sqlDataTypeName.ToLower())
    {
        case "bigint":
            return "Int64";
        case "binary":
        case "image":
        case "varbinary":
            return "byte[]";
        case "bit":
            return "bool";
        case "char":
            return "char";
        case "datetime":
        case "smalldatetime":
            return "DateTime";
        case "decimal":
        case "money":
        case "numeric":
            return "decimal";
        case "float":
            return "double";
        case "int":
            return "int";
        case "nchar":
        case "nvarchar":
        case "text":
        case "varchar":
        case "xml":
            return "string";
        case "real":
            return "single";
        case "smallint":
            return "Int16";
        case "tinyint":
            return "byte";
        case "uniqueidentifier":
            return "Guid";

        default:
            return null;
    }
}
```