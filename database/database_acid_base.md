# 資料庫 ACID 特性、BASE 原則

## ACID

- Atomicity: Atomicity means that you guarantee that either all of the transaction succeeds or none of it does. You don't get part of it succeeding and part of it not. If one part of the transaction fails, the whole transaction fails. With atomicity, it's either "all or nothing". 該交易如果包含多個步驟，所有步驟要嘛全都成功，要嘛全都失敗。

- Consistency: This ensures that you guarantee that all data will be consistent. All data will be valid according to all defined rules, including any constraints, cascades, and triggers that have been applied on the database. 資料一致性，交易在修改前或修改後皆維持資料一致性，意即不會違反本來就定義在 schema 的規則，例：條件約束、外部索引鍵、觸發程序。

- Isolation: Guarantees that all transactions will occur in isolation. No transaction will be affected by any other transaction. So a transaction cannot read data from any other transaction that has not yet completed. 各個交易之間不會互相干擾。

- Durability: Durability means that, once a transaction is committed, it will remain in the system – even if there's a system crash immediately following the transaction. Any changes from the transaction must be stored permanently. If the system tells the user that the transaction has succeeded, the transaction must have, in fact, succeeded. 在交易成功後，即使遇到系統當機或重開，已成功的交易的資料也不會遺失。

---

## BASE
遵守 ACID 原則會讓吞吐量下降很多，對有些產業需要高併發和低回應時間的情形就難以接受。

因此有人提出將 ACID 原則放寬，提出 BASE 原則

- Basically Available：保持服務基本可用。

- Soft State: 狀態可以有一段時間的不同步。

- Eventual consistency：最終一致性，雖然有一段時間不同步，但追求最後結果一致。