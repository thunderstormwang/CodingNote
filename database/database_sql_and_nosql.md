# 資料庫 SQL VS NoSQL

## SQL
- 有 ACID 特性 [資料庫 ACID 特性、BASE 原則](database_acid_base.md)

- 資料表需預先設定架構(schema)，資料表之間的關係須先定義好

- 資料可能橫跨好幾個 Table，可以使用 JOIN 來連結多個資料表，取出需要的資料

- 承上，較適合更新頻繁的情境，讀寫的速度受到 join 的限制，但如果資料表足夠正規化，更新只須對少數 Table 做更新

- 不容易/很難做水平擴展

---  

## NoSQL(Not only SQL，以 MongoDB 為例)
- 有 BASE 原則 [資料庫 ACID 特性、BASE 原則](database_acid_base.md)

- 不需要事先定義好資料的 schema 以及資料之間的關聯，可由定義資料文件(document)的結構，也可以自由新增欄位，不需要回去修改過去的資料文件(document)

- 資料只須儲存在少數幾個 colletion，不用做 join 就拿到所需的資料

- 承上，較適合讀寫頻繁的情境，讀寫不受到 join 的限制，非常快速，但在更新時須對所有含有相關資料的 collection 逐一做更新

- 較容易做水平擴展
  