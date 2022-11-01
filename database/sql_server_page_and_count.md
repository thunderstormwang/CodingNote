# Sql Server 分頁且回傳總筆數

常常有需求是要將列表分頁，且回傳總筆數

## 分兩次查
```sql
DECLARE @pageSize INT = 10; 
DECLARE @pageIndex INT = 1; 
DECLARE @startTime DATETIME = '2022-10-28 00:00:00.000'; 
DECLARE @endTime DATETIME = '2022-10-29 00:00:00.000' 

SELECT TGCode,
       TSCode,
       Third_Code,
       Create_Time
FROM EGB_SYNC_UAT.dbo.EB_File_Pay efp
WHERE Create_Time BETWEEN @startTime AND @endTime
ORDER BY efp.Create_Time ASC 
OFFSET (@pageIndex - 1) * @pageSize ROWS FETCH NEXT @pageSize ROWS ONLY;

SELECT Count(1) AS TotalCount
FROM EGB_SYNC_UAT.dbo.EB_File_Pay efp
WHERE Create_Time BETWEEN @startTime AND @endTime
```

如果想一次就拿回分頁結果和總筆數，可以用以下方法

## Using COUNT(*) OVER() 

```sql
DECLARE @pageSize INT = 10; 
DECLARE @pageIndex INT = 1; 
DECLARE @startTime DATETIME = '2022-10-28 00:00:00.000'; 
DECLARE @endTime DATETIME = '2022-10-29 00:00:00.000' 

SELECT TGCode,
       TSCode,
       Third_Code,
       Create_Time,
       [TotalCount] = COUNT(*) OVER ()
FROM EGB_SYNC_UAT.dbo.EB_File_Pay efp
WHERE Create_Time BETWEEN @startTime AND @endTime
ORDER BY efp.Create_Time ASC OFFSET (@pageIndex - 1) * @pageSize ROWS FETCH NEXT @pageSize ROWS ONLY;
```

## Using Common Table Expression 

```sql
DECLARE @pageSize INT = 10; 
DECLARE @pageIndex INT = 1; 
DECLARE @startTime DATETIME = '2022-10-28 00:00:00.000'; 
DECLARE @endTime DATETIME = '2022-10-29 00:00:00.000' 

WITH main_cte
AS (SELECT TGCode,
           TSCode,
           Third_Code,
           Create_Time
    FROM EGB_SYNC_UAT.dbo.EB_File_Pay efp
    WHERE Create_Time BETWEEN @startTime AND @endTime
   ),
     count_cte
AS (SELECT COUNT(1) AS TotalCount
    FROM main_cte
   )
SELECT *
FROM main_cte,
     count_cte
ORDER BY main_cte.Create_Time ASC 
OFFSET (@pageIndex - 1) * @pageSize ROWS FETCH NEXT @pageSize ROWS ONLY;

```

## Using Cross Apply 

```sql
DECLARE @pageSize INT = 10; 
DECLARE @pageIndex INT = 1; 
DECLARE @startTime DATETIME = '2022-10-28 00:00:00.000'; 
DECLARE @endTime DATETIME = '2022-10-29 00:00:00.000' 

SELECT TGCode,
       TSCode,
       Third_Code,
       Create_Time,
       Cnt.TotalCount
FROM EGB_SYNC_UAT.dbo.EB_File_Pay efp
    CROSS APPLY
(
    SELECT COUNT(1) AS TotalCount
    FROM EGB_SYNC_UAT.dbo.EB_File_Pay efp
    WHERE Create_Time BETWEEN @startTime AND @endTime
) AS Cnt
WHERE Create_Time BETWEEN @startTime AND @endTime
ORDER BY efp.Create_Time ASC 
OFFSET (@pageIndex - 1) * @pageSize ROWS FETCH NEXT @pageSize ROWS ONLY;
```

---

參考自 [SQL SERVER – How to get total row count from OFFSET / FETCH NEXT (Paging)](https://raresql.com/2015/03/30/sql-server-how-to-get-total-row-count-from-offset-fetch-next-paging/)，用 cte 感覺較簡潔，因為兜出來的 table 和 where 條件只要寫一次，其它兩種方法都要重覆一次。
至於效能方面，我手邊沒資料可以試，就參考原作的吧