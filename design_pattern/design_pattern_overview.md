# 設計模式概觀

##　設計模式的用途
+ 設計模式是被用來解決特定的需求
+ 如何在不重新設計下進行改變
+ 組合多種設計模式
+ 為何使用設計模式
  + 明確的職責與意圖
  + 清晰的程式碼
  + 良好的封裝性
  + 靈活的擴充性

<br/>

## 設計觀點的介面
+ 設計觀點的介面並不僅僅是 Interface 型別。
+ 物件宣告的每一個操作都具有簽章。
+ 一個物件定義的所有簽章就被稱為介面。
+ 依賴介面設計，而不要倚賴實作

## 設計模式的分類
+ 生成模式
  + 將物件建構的過程加以抽象化，使系統能夠獨立於物件的建立、組成與表現方式之外。
  + 將創造與管理物件的職責獨立。
+ 結構模式
  + 透過組合類別或是物件來產生更大的結構，用以適應更高層次的邏輯需求。
  + 類別結構模式 (如:Class Adapter) 的目的是組合出外部介面和內部實作。
  + 物件結構模式則是注重在組合多個物件來實現新的功能。
+ 行為模式
  + 行為模式的重點在於物件之間的演算法與權責分配的問題。
  + 這些模式不僅是在描述單一個物件或類別的模式，同時也在描述多個物件或類別間如何執行通訊。


| 生成模式         | 結構模式  | 行為模式        |
|------------------|-----------|-----------------|
| Factory Method   | Adapter   | Interpreter     |
| Abstract Factory | Bridge    | Template Method |
| Builder          | Composite | Command         |
| Prototype        | Decorator | Iterator        |
| Singleton        | Façade    | Mediator        |
|                  | Flyweight | Memento         |
|                  | Proxy     | Observer        |
|                  |           | State           |
|                  |           | Strategy        |
|                  |           | Chain of        |
|                  |           | Responsibility  |

---

[多用組合，少用繼承](composition_inheritance.md)