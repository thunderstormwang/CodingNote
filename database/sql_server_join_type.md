# SQL Server 的 Join 類型

這裡指的是 
- Nsteed Loop
- Merge Join
- Hash Match

假如左表有 N 行資料，右表有 M 行資料

## Nested Loop
- Complexity: O(N * M)
- 當兩個 Table 中，其中一個 Table 數量明顯較小，且較大的 Table 有建立 index 在 join 條件上，就可能採用 Nested Loop
- 在其中一個 Table 數量較小會很有效率，用 Nested Loop 會較快。例如外層迴圈只有 5 列資料，那內層迴圈只要做 5 次 index 的查詢就可以找到所有資料。

## Merge Join
- Complexity: O(N + M)
- 在大型 Table 且已依照 join 條件排序過時，此方法會很有效率
- 在 join 條件也須使用 等號
- 兩個 Table 各有個指針從第一個資料列開始比較，因為已排序過，當兩邊資料不相符時，就移動資料較小的指針。
- 因為可能需要排序，所以會耗費較多資源。

## Hash Match
- Complexity: O(N * hc + M * hm + J) or O(N + M) if you ignore resource consumption costs
  - hc 是指建立 hash table 的複雜度
  - hm 是指 hasm match function 的複雜度
  - J s a “joker” complexity addition for the dynamic calculation and creation of the hash function
  - 因為 hc, hm, J 通常較小，所以仍寫作 O(N + M)
- Last-resort join type
- Uses a hash table and a dynamic hash match function to match rows
- Higher cost in terms of memory consumption and disk IO utilization
- 當 index 沒建立在 join 條件上就可能採用此方法
- 較耗效能，因為會用到 cpu 計算 hash，且用 memory 存 hash table，如果 memory 不夠，會用到 disk IO。