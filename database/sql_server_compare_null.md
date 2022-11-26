# SQL Server 比較 Null

Null(空值) 在 SQL 裡代表一個未知的狀態，它不等於任何值，也無法被直接比較

<b>跟一般程式語言不一樣，它無法被比較，用 <>、!= 或 = 去比較都會得到 False<b>

以下皆不會印出 1
```sql
SELECT 1 
WHERE NOT (100 = NULL)
```
```sql
SELECT 1 
WHERE 100 != NULL
```
```sql
SELECT 1 
WHERE 100 > NULL
```
```sql
SELECT 1 
WHERE 100 < NULL
```
```sql
SELECT 1 
WHERE 100 NOT IN (NULL)
```

<br/>需要指定當遇到 NULL 時該怎麼辦
```sql
SELECT 1 
WHERE 100 IS NOT NULL
```
```sql
SELECT 1 
WHERE 100 != ISNULL(NULL, 0)
```
```sql
SELECT 1 
WHERE 100 NOT IN (ISNULL(NULL, 0))
```
>1