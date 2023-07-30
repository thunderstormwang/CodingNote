# Quick Sort

是一種 Divide and Conquer 的排序方式

![quick sort](./imgs/quicksort.png)

選一個基準點(pivot)，比 pivot 小的放左邊，大的放右邊，  
再依 pivot 分成兩個 partition，  
然後重覆上述動作


C# 程式碼  
寫的時候要注意陣列有無超過邊界  
當 right 比 left 還小時，就代表 right 指向從左邊數來第一個 小於等於 pivot 的元素，也就是找到可跟 pivot 交換的位置
```csharp
public Main()
{
    var nums = new int [] { 3, 2, 1, 5, 6, 4 };
    QuickSort(nums, 0, nums.Length - 1);
}

private void QuickSort(int[] nums, int start, int end)
{
    if (start > end)
    {
        return;
    }
                
    var pivot = start;
    var left = start + 1;
    var right = end
    while (left <= right)
    {
        while (right >= pivot && nums[right] > nums[pivot])
        {
            right--;
        }
        
        while (left <= end && nums[left] <= nums[pivot])
        {
            left++;
        }
        
        if (left < right)
        {
            (nums[left], nums[right]) = (nums[right], nums[left]);
        }
    }
                
    (nums[pivot], nums[right]) = (nums[right], nums[pivot])            
    QuickSort(nums, start, right - 1);
    QuickSort(nums, right + 1, end);
}
```
