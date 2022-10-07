# Elasticsearch

## 欄位型態 text 和 keyword 的差別

keyword 適合用來存結構化的內容，如 ID、email 地址、域名、狀態碼、郵遞區號。

適合用來排序、聚合、term-level query。

整個內容不會被分詞器分割就作為倒排索引

<br/>text 適合用來存非結構化的內容，如 email 內容、商品說明當儲存的欄位
```
{ "message": "Hello World" }
```
會被分詞器分成「Hello」、「World」做為倒排索引

---

## Term Query & Match Query & Match Phrase Query

### Term Query
適合用來搜尋型態 keyword 的欄位，要完全符合輸入才會被搜尋出來。

Match Query 和 Match Phrase Query 的輸入都會被分詞器處理過再拿去搜尋

### Match Query
輸入「Hello World」會被分成「Hello」、「World」

然後以下皆符合結果
- 「I just said hello world」
- 「Hello world」
- 「World Hello」
- 「Hello dear world」

### Match Phrase Query
輸入「Hello World」也會被分成「Hello」、「World」但多了些條件
- 所有詞都須出現
- 詞的出現順序要跟輸入一樣
- 中間不能有其它詞

所以只有第 1 跟第 2 才符合結果
- 「I just said hello world」
- 「Hello world」
- 「World Hello」
- 「Hello dear world」

---

.Net 在 elasticsearch 可用兩種套件，[Elasticsearch.Net](https://github.com/NickCraver/NEST/tree/master/src/Elasticsearch.Net) 和 [NEST](https://github.com/NickCraver/NEST)，[兩者差別](https://github.com/NickCraver/NEST)