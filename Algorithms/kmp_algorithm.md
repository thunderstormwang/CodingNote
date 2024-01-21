# KMP Algorithm - Knuth-Morris-Pratt Algorithm

比對字串 text 是否有出現與字串 pattern 相同的字串

暴力法的時間複雜度為 O(mn)，KMP 算法的時間複雜度為 O(m+n)。

KMP 需要先建立一個部分匹配表，LPS，Longest Prefix Suffixe，這個表的內容是 pattern 字串中，每個字元前面的字串中，有多少個字元是相同的。

假如 pattern 為 `ABCABCABD`

可以建立一個 LPS 為 

 | 字串  | A | B | C | A | B | C | A | B | D |
 |-------|---|---|---|---|---|---|---|---|---|
 | value | 0 | 0 | 0 | 1 | 2 | 3 | 4 | 5 | 0 |

其精神為，如果我們比對 text[8] `D`失敗，我們可以從一個位置 lps[7] 知道前面有 5 個字元是相同的，那就退回從 text[5] `C` 開始比對，如果也不是，那就依據 lps[4]，前面有 2 個字元是相同的，退回 text[2]，如果也不是，lps[2] 值為 0，那就是重頭開始比對。

建立 lps 的程式碼如下
    
```csharp
private int[] GetLpsArray(string text)
{
    var result = new int [text.Length];
    var i = 1;
    var pre = 0;

    while (i < text.Length)
    {
        if (text[i] == text[pre])
        {
            result[i] = pre + 1;
            pre++;
            i++;
        }
        else if (pre > 0)
        {
            pre = result[pre - 1];
        }
        else
        {
            i++;
        }
    }
    return result;
}
```

比對的程式碼如下

```csharp
public int Compare(string text, string pattern)
{
    if (text.Length < pattern.Length)
    {
        return -1;
    }
    
    var lpsArray = GetLpsArray(pattern);
    var i = 0;
    var j = 0;
    while (i < text.Length)
    {
        if (text[i] == pattern[j])
        {
            i++;
            j++;
        }
        else if (j > 0)
        {
            j = lpsArray[j - 1];
        }
        else
        {
            i++;
        }
        if (j == pattern.Length)
        {
            return i - j;
        }
    }

    return -1;
}
```