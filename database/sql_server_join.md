# SQL Server Apply 語法

使用以下腳本建立 table 和初始資料
```sql
CREATE TABLE [dbo].[MyDepartment](
	[Id] [int] NOT NULL IDENTITY,
	[DepartmentId] [int] NOT NULL,
	[Name] [varchar](250) NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX  = OFF, STATISTICS_NORECOMPUTE  = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS  = ON, ALLOW_PAGE_LOCKS  = ON) ON [PRIMARY]
) ON [PRIMARY]
GO

INSERT [dbo].[MyDepartment] ([DepartmentId], [Name]) VALUES (1, N'Engineering')
INSERT [dbo].[MyDepartment] ([DepartmentId], [Name]) VALUES (2, N'Administration')
INSERT [dbo].[MyDepartment] ([DepartmentId], [Name]) VALUES (3, N'Sales')
INSERT [dbo].[MyDepartment] ([DepartmentId], [Name]) VALUES (4, N'Marketing')
INSERT [dbo].[MyDepartment] ([DepartmentId], [Name]) VALUES (5, N'Finance')
GO

CREATE TABLE [dbo].[MyEmployee](
	[Id] [int] NOT NULL IDENTITY,
	[Name] [varchar](250) NOT NULL,
	[DepartmentId] [int] NOT NULL,
	[Age] [int] NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX  = OFF, STATISTICS_NORECOMPUTE  = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS  = ON, ALLOW_PAGE_LOCKS  = ON) ON [PRIMARY]
) ON [PRIMARY]
GO

INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'Assassin', 1, 25)
INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'Batman', 2, 30)
INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'Shadow of Mordor', 3, 28)
INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'War of Mordor', 3, 27)
INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'Wrath of Lick King', 4, 48)
INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'Mist of Pandarian', 4, 47)
INSERT [dbo].[MyEmployee] ([Name], [DepartmentID], [Age]) VALUES (N'Dragonflight', 4, 36)
GO
```

以下的寫法，都可以列出每個部門底下年紀最大的兩個員工，但第二個寫法比第一個好，雖然都用了子查詢，第一句的子查詢在 MyDepartment 的每一行都要跑一次，資料量一大可能被拖慢，反之，第二句的子查詢則需跑一次即可  

子查詢會跑多次
```sql
SELECT D.DepartmentId, 
       D.Name AS DepartmentName, 
       E.Id AS EmployeeId, 
       E.Name AS EmployeeName, 
       E.Age
FROM MyDepartment D
LEFT JOIN MyEmployee E
ON E.DepartmentId IN (
	SELECT TOP 2 Id
	FROM MyEmployee E
	WHERE E.DepartmentId = D.Id
	Order By E.ID DESC
)
WHERE E.Id IS NOT NULL
GO
```
  
子查詢只會跑一次
```sql
SELECT D.DepartmentId, 
       D.Name AS DepartmentName, 
       E.Id AS EmployeeId, 
       E.Name AS EmployeeName, 
       E.Age
FROM MyDepartment D
INNER JOIN (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY DepartmentId ORDER BY Age DESC) AS Row 
    FROM MyEmployee
    ) E
ON D.DepartmentID = E.DepartmentID
WHERE E.Row <= 2
GO
```