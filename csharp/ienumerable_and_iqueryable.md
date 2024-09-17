# [C# 3.0]IEnumerable v.s IQueryable

IEnumerable
- queries in-memory data
- 在 System.Collections 命名空間

IQueryable
- queries out-of-memory data stores
- 在 System.LINQ 命名空間
- 繼承自 IEnumerable

<br/>在使用 Entity Framework 的時候

<br/>若轉成 IEnumerable
```csharp
IEnumerable<Customer> customers = _context.Customers.AsEnumerable().Where(c => c.IsActive);
```
組出來的 sql
```sql
SELECT * FROM [Customers] AS [c]
```

<br/>若轉成 IQueryable
```csharp
IQueryable<Customer> customers = _context.Customers.Where(c => c.IsActive);
```
組出來的 sql
```sql
SELECT * FROM [Customers] AS [c] WHERE [c].[IsActive] == 1 
```

<br/>就如字面上的意義，當你查詢的對象不是記憶體時，把所有資料都載入的記憶體再做篩選就顯得很不切實際
