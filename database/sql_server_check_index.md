# 查詢索引 Check Index 

列出該索引包含的 index key
```sql
--列出該索引包含的 index key
DECLARE @tableName VARCHAR(50)= 'MainDB.dbo.MyTable';
--[SCHEMA-NAME.TABLE-NAME]

EXEC sp_helpindex
     @tableName;
```

列出該索引是 Cluster/NonCluster、是否唯一
```sql
--列出該索引是 Cluster/NonCluster、是否唯一
DECLARE @tableName VARCHAR(50)= 'MainDB.dbo.MyTable';
--[SCHEMA-NAME.TABLE-NAME]

SELECT name AS Index_Name,
       type_desc AS Index_Type,
       is_unique,
       OBJECT_NAME(object_id) AS Table_Name
FROM sys.indexes
WHERE is_hypothetical = 0
      AND index_id != 0
      AND object_id = OBJECT_ID(@tableName);
```

列出該索引的 index key, inlcuded column
```sql
--列出該索引的 index key, inlcuded column
DECLARE @tableName VARCHAR(50)= 'MainDB.dbo.MyTable';
--[SCHEMA-NAME.TABLE-NAME]

SELECT a.name AS Index_Name,
       OBJECT_NAME(a.object_id),
       COL_NAME(b.object_id, b.column_id) AS Column_Name,
       b.index_column_id,
       b.key_ordinal,
       b.is_included_column
FROM sys.indexes AS a
     INNER JOIN sys.index_columns AS b
     ON a.object_id = b.object_id
        AND a.index_id = b.index_id
WHERE a.is_hypothetical = 0
      AND a.object_id = OBJECT_ID(@tableName);
```