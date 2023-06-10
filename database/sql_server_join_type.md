# SQL Server 的 Join 類型

這裡指的是 
- Nsteed Loop
- Merge Join
- Hash Match

假如左表有 N 行資料，右表有 M 行資料

## Nested Loop
- Complexity: O(N * logM)
- Used usually when one table is significantly small
- The larger table has an index which allows seeking it using the join key

## Merge Join
- Complexity: O(N + M)
- Both inputs are sorted on the join key
- An equality operator is used
- Excellent for very large tables

## Hash Match
- Complexity: O(N * hc + M * hm + J) or O(N + M) if you ignore resource consumption costs
  - hc 是指建立 hash table 的複雜度
  - hm 是指 hasm match function 的複雜度
  - J s a “joker” complexity addition for the dynamic calculation and creation of the hash function
  - 因為 hc, hm, J 通常較小，所以仍寫作 O(N + M)
- Last-resort join type
- Uses a hash table and a dynamic hash match function to match rows
- Higher cost in terms of memory consumption and disk IO utilization