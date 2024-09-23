# Binary Search

情境：不確定要找的數有沒有存在，存在就回傳位置，不存在就回傳第一個大於目標的位置

```csharp
var array = new int[] { 0, 1, 2, 3, 5, 6, 7, 8, 9, 10 };
var target = 4;

if (target < array[0]) 
{
    Console.WriteLine("比所有的元素都小");
    return;
}

if (target > array[array.Length - 1])
{
    Console.WriteLine($"比所有的元素都大");
    return;
}
        
var result = BinarySearch(array, target);

if (array[result] == target)
{
    Console.WriteLine($"found: {result}");
}
else
{
    Console.WriteLine($"not found, insert at: {result}");
}
```

```csharp
private static int BinarySearch(int[] array, int target)
{
    var low = 0;
    var high = array.Length - 1;

    while (low <= high)
    {
        var mid = low + (high - low) / 2;

        if (array[mid] == target)
        {
            return mid;
        }

        if (array[mid] < target)
        {
            // 繼續在右半部分搜尋
            low = mid + 1;
        }
        else
        {
            // 縮小到左半部分搜尋
            high = mid - 1;
        }
    }
        
    // low: 插入位置
    // high: 插入位置 - 1

    return low;
}
```

若目標不存在，，會有以下過程：  

| 僅次於目標的數 | 第一個大於目標的數 |
|----------------|--------------------|
| low            | high               |

某次迴圈會讓 low 和 high 差 1，如上表所示，  

再下次迴圈，計算 mid = low  
array[mid] < target，所以 low = mid + 1  

| 僅次於目標的數 | 第一個大於目標的數 |
|----------------|--------------------|
|                | low, high          |

再下次迴圈，計算 mid = low = high  
array[mid] > target，所以 high = mid - 1  

| 僅次於目標的數 | 第一個大於目標的數 |
|----------------|--------------------|
| high           | low                |

再下次迴圈，不符合條件而跳出迴圈，  
此時 low 會指向第一個大於目標的數的位置，high 會指向 僅次於目標的數的位置