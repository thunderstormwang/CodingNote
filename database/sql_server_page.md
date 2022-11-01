# SQL Server 分頁語法

## SQL Server 2012 以後
- 使用 Offset Fetch 語法
- 需搭配 Order By
- Offset：略過前面幾列資料
- Fetch Next：取接下來幾列資料

```sql
DECLARE @intPage INT= 1;
DECLARE @intPageSize INT= 1000;

SELECT *
FROM dbo.MainAccount_Turnover mat
ORDER BY mat.CreateTime DESC
OFFSET(@intPage - 1) * @intPageSize ROW FETCH NEXT @intPageSize ROW ONLY;
```

## SQL Server 2008 以前
- 使用 Row_Number 語法

```sql
DECLARE @intPage INT = 1;
DECLARE @intPageSize INT = 1000;

SELECT *
FROM
(
    SELECT ROW_NUMBER() OVER (ORDER BY mat.CreateTime DESC) AS RowNumber,
           *
    FROM MainAccount_Turnover mat
) TableResult
WHERE TableResult.RowNumber BETWEEN (@intPage - 1) * @intPageSize AND @intPage * @intPageSize;
```

<br/>可以搭配 Partition By 做分組，例如取每組前 1000
```sql
DECLARE @intPage INT= 1;
DECLARE @intPageSize INT= 1000;

SELECT *
FROM
(
    SELECT ROW_NUMBER() OVER(PARTITION BY mat.PlatformCode ORDER BY mat.CreateTime  DESC) AS RowNumber,
           *
    FROM MainAccount_Turnover mat
) TableResult
WHERE TableResult.RowNumber BETWEEN(@intPage - 1) * @intPageSize AND @intPage * @intPageSize;
```
