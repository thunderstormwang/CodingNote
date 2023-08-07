# Heap Sort

分兩個步驟
- 將陣列轉為 max heap(這過程也稱作 heapify)，也就是每個父節點都大於子節點
- 將陣列第一個與最後一個交換值，然後將 heap 的長度減 1，再對陣列第一個做 heapify，讓它到應該在的位置

不需要額外的空間  
存取的位置不是連續，較不利 CPU 快取

C# 程式碼  
```csharp
public Main()
{
    var nums = new int [] { 3, 2, 1, 5, 6, 4 };
    HeapSort(nums, 0, nums.Length - 1);
}

public void HeapSort(int[] nums)
{
    // build max heap
    for (var i = nums.Length / 2 - 1; i >= 0; i--)
    {
        Heapify(nums, i, nums.Length - 1);
                
    for (var i = nums.Length - 1; i >= 0; i--)
    {
        (nums[0], nums[i]) = (nums[i], nums[0]);
        Heapify(nums, 0, i - 1);
    }
}

private void Heapify(int[] nums, int i, int end)
{
    var leftIndex = 2 * i + 1;
    var rightIndex = 2 * i + 2;
    var largestIndex = i            
    
    if (leftIndex <= end && nums[leftIndex] > nums[largestIndex])
    {
        largestIndex = leftIndex;
    }
                
    if (rightIndex <= end && nums[rightIndex] > nums[largestIndex])
    {
        largestIndex = rightIndex;
    }
    
    if (largestIndex == i)
    {
        return;
    }
                
    (nums[largestIndex], nums[i]) = (nums[i], nums[largestIndex]);
    Heapify(nums, largestIndex, end);
}
```