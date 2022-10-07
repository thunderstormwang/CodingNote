# ELK 概觀

ELK 是由三個工具組成，Elasticsearch (E) 、Logstash (L) 、Kibana (K) 組成的Log 過濾、儲存、分析、視覺化系統，另外我們還需要一個叫Beat的工具來擔任Log的採集工具。

- ElasticSearch 是整體系統的核心部分，是一個非關聯式資料庫以JSON格式儲存資料，擁有一個非常厲害的索引功能，默認為9200 port
- Logstash是一個過濾器(不過其實本身也可以作為採集器)，會把所有採集到的Log傳遞到logstash中。logstash可以依照規則將Log分割並傳遞到ElasticSearch (文章之後用ES表示)，默認是5044 port。
- Kibana 是圖形化展示工具，將儲存在ES內的資料用各式各樣圖表呈現出來，默認是5601 port。
- Beats 是Log採集工具，負責將本台電腦的Log檔取出傳遞到logstash做過濾，或是直接傳遞到ES儲存器起來。

<br/>整體流程如下:

![elk 概觀](imgs\elk_overriew.png)

---

參考自
- [ELK 實作分散式log採集系統](https://lufor129.medium.com/elk-%E5%AF%A6%E4%BD%9C%E5%88%86%E6%95%A3%E5%BC%8Flog%E6%8E%A1%E9%9B%86%E7%B3%BB%E7%B5%B1-d3e729624af4)