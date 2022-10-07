# Elasticsearch Query DSL

Leaf query clause，最末端的查詢句
- Match
- MatchPhrase
- Term
- Range

<br/>Compound query clause，複合式查詢句，可以是其它複合式查詢句和 Leaf query clause 的組合

Compound query 有以下幾種
- bool query → 只有這個看得懂...其它應該現階段用不到
- boosting query
- constant_score query
- dis_max query
- function_score query

<br/>bool query 可以使用以下種類
- must
- filter → 單純過濾查出來的文件，不會影響 _score
- should
- must_not

符合的條件越多，_score 越高，排序也就越前面