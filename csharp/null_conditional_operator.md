# [C# 6.0]?. & ?[ Operator - Null-conditional Operator

C# 6.0 推出了新寫法，簡化物件的 null 檢查

以下寫法
```csharp
List<int> intList = null;

int? count = intList?.Count;
```

compiler 幫你轉成以下寫法

```csharp
List<int> intList = null;

int? count = null;

if  (intList == null)
{
    count = null;
}
else
{
    count = intList.Count;
}

```

<br/>以下寫法
```csharp
public int GetCount(List<int> intList)
{
    return intList?.Count;
}
```

compiler 幫你轉成以下寫法

```csharp
public int GetCount(List<int> intList)
{
    if (intList == null)
    {
        return null;
    }
    else
    {
        return intList.Count;
    }
}
```

<br/>以下寫法
```csharp
ClassRoom class= null;

string result = classRoom?.Teacher?.Name;
```

compiler 幫你轉成以下寫法

```csharp
ClassRoom class= null;

string result = string.Empty;
if (classRoom == null || classRoom.Teacher == null)
{
    result = null;
}
else
{
    result = classRoom.Teacher.Name;
}
```

<br/>也可以用在 indexer

```csharp
List<ClassRoom> list = null;

string name = list?[0].Name;
```

<br/>與 ?? 運算子結合使用

```csharp
List<ClassRoom> list = null;

int count = list?.Count ?? 0;
```