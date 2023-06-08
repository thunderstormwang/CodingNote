# CluIndex

## Primary Key

- table 中每一個 row 的唯一識別值
- 同個 table 只能設定一組由一個或多個欄位去組成 Primary Key，不過也不一定要設定 Primary Key
- 資料在整個 table 內不能重覆
- 欄位值不可為 null
- 本身就是一個 index，在 Sql Server，如果建立 table 時，沒有建立其它 index，則 Primary Key 會被預設為 Clustered Index

## Clustered Index

- 每個 Table 只能有一個 Clustered Index，且 Table 資料就會依據 Clustered Index 的方式排列，這好比書籍的目錄，每本書只能有一個目錄，因為整本書會依照這個目錄排序。
- Clustered Index 可指定在整個 Table 是否唯一，欄位也可以有 null
- Clustered Index 不必然是 Primary Key
- Clustered Index 在建立就是一個 B+ Tree，資料則放在葉節點
- 如果欄位經常被更新，就不適合當作 index，減少維護 B+ Tree 的成本
- 不會有額外的 Key Lookup(索引鍵查閱)

## Non-Clustered Index

- 每個 Table 能有許多 Non-Clustered Index，這好比書集後面的索引目錄，每本書可以有很多種索引目錄，例如依照字母排序、依照附錄A、依照附錄B。
- 每個 Non-Clustered Index 都是獨立的 B+ Tree，在葉節點存放指標，指向 Clustered Index
- 如果欄位經常被更新，就不適合當作 index，減少維護 B+ Tree 的成本
- 可能會有額外的 Key Lookup(索引鍵查閱)

## Key Lookup(索引鍵查閱)

Non-Clustered index 是由 Index Key、Included Column 和 Clustered Key 組成。當利用 Non-Clustered Index 搜尋資料時，會先利用 Index Key 來搜尋符合條件的所有資料：

情況一：

```sql
Select [CustomerID], [Address] 
From Customer 
Where CustomerID < 100
```
Non-Clustered Index 欄位為 CustomerID

假如利用這些資料的 Included Column 仍不滿足所需的欄位，就會進一步利用 Clustered Key 進入 Clustered Index 內去搜尋所需要資料，此行為稱為索引鍵查閱(Key Lookup)，2005 SP2 之前稱為書籤查詢(Bookmark Lookup)，也就是說要做兩次查詢，會增加鎖定和執行時間，此情況應避免。

情況二：
假如利用這些資料的 Included Column 就能帶出所有需要的資料欄位，此情況稱為 Covering Index(覆蓋索引)

增加 Included Column 會增加查詢效率，但會減少寫入效率，因為 Non-Clustered Index 是另外一個 B+ Tree。  
資料庫寫入的欄位若不包含在 Non-Clustered Index 的欄位內，則寫入時就不用去維護該 Non-Clustered Index 的 B+ Tree。  
在寫入該欄位時也可同時進行對該 Non-Clustered Index 的讀取