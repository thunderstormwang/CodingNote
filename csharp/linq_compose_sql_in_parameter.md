# [C# 3.0]用 Linq 組出 sql 句裡 IN ( ) 內的參數

我們有個陣列，內容數量不固定
```csharp
var tags = new List<string>() { "ruby", "rails", "scruffy", "rubyonrails" };
```

想組出下列 sql 句
```sql
SELECT * FROM [dbo].[Tags]
WHERE [Tags] IN ('ruby', 'rails', 'scruffy', 'rubyonrails')
```

可用 Linq 做到
```csharp
var tags = new List<string>() { "ruby", "rails", "scruffy", "rubyonrails" };
var cmdText = "SELECT * FROM [dbo].[Tags] WHERE [Tags] IN ({0})";
var parameters = string.Join(", ", tags.Select((tag, index) => $"@tag{index}"));

cmdText = string.Format(cmdText, parameters);

using(var conn = new SqlConnection())
{
    using(var cmd = new SqlCommand(cmdText, conn))
    {
        for (int i = 0; i < tags.Count; i++)
        {
            cmd.Parameters.AddWithValue($"@tag{i}", tags[i]);
        }

        // ...
    }
}
```