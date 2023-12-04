# [C# 2.0]TransactionScope

以前的寫法，需要先呼叫 SqlConnection.BeginTransaction，然後依執行結果再判斷該呼叫 SqlTransaction.Commit 或 SqlTransaction.RollBack)

C# 2.0 後提供 TransactionScope，寫法可較簡潔。  

```csharp
public static void Main(string[] args)
{
    try
    {
        using (TransactionScope scope = new TransactionScope())
        {
            using (SqlConnection connection = new SqlConnection())
            {
                connection.Open();
                using (SqlCommand command = new SqlCommand())
                {
                    command.ExecuteNonQuery();
                }
            }
            
            // The Complete method commits the transaction. If an exception hasbeen thrown,
            // Complete is not  called and the transaction is rolled back.
            scope.Complete();
        }
    }
    catch (TransactionAbortedException ex)
    {
        Console.WriteLine(ex);
    }
}
```