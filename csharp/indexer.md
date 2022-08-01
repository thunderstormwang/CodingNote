# [C#]Indexer - 索引子

```csharp
public class SampleCollection<T>
{
   // Declare an array to store the data elements.
   private T[] arr = new T[100];

   // Define the indexer to allow client code to use [] notation.
   public T this[int i]
   {
      get { return arr[i]; }
      set { arr[i] = value; }
   }
}
```

```csharp
var stringCollection = new SampleCollection<string>();      stringCollection[0] = "Hello, World";
Console.WriteLine(stringCollection[0]);
```
>Hello, World.

<br/>索引子概觀
- 索引子可以讓物件利用與陣列類似的方式編製索引。
- get 存取子會傳回值。 set 存取子會指派值。
- this 關鍵字是用來定義索引子。
- value 關鍵字是用來定義 存取子要指派的值。
- 索引子不一定要由整數值編製索引。您可自行決定如何定義特定的查詢機制。
- 索引子可以多載。
- 索引子可具有一個以上的型式參數，例如，存取二維陣列時。