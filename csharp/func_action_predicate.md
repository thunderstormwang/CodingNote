# [C# 3.0]Func、Action、Predicate

Linq 很常見到 Func、Action、Predicate 三種之一的委派

## Func<TResult> 委派
依照它的定義, 是一個泛型委派(generic delegate)，一個不用任何參數，且傳回泛型型別 TResult 的委派。
<br/>linq 的 Select 就是這種作法
```csharp
public delegate TResult Func<out TResult>()
```

#### 用 C# 1.0 的 delegate:
宣告具有同樣簽章(signature)的委派
```csharp
public delegate TResult myDelegate<TResult>();
```

<br/>宣告符合該簽章的函式
```csharp
public string string Output()
{
    return "Hello World";
}
```

<br/>以該委派去達到同樣效果
```csharp
myDelegate<string> myFuncDel = Output;
Console.WriteLine(myFuncDel());
```
>Hello World

#### 用 C# 2.0 的 Anonymous Method:
```csharp
myDelegate<string> myFuncDel = new myDelegate<string>(delegate(){
    return "Hello World";
});
Console.WriteLine(funcDel());
```
>Hello World

<br>看到這裡, 大概看得出來 Func 就是幫你先把委派定義好, 少了宣告 delegate 的動作
```csharp
Func<string> myFuncDel = Output;
Console.WriteLine(myFuncDel());
```
>Hello World

<br/>結合 lambda
```csharp
Func<string> myFuncDel = () => "Hello World";
Console.WriteLine(myFuncDel());
```
>Hello World


## Func<T1, T2, TResult>  委派
依照它的定義, 是一個泛型委派(generic delegate)，一個有2個參數為泛型型別 T1、T2，且傳回泛型型別 TResult 的委派
<br/>linq 的 Select 就是這種作法
```csharp
public delegate TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2)
```

#### 用 C# 1.0 的 delegate:
宣告具有同樣簽章(signature)的委派
```csharp
public delegate TResult myDelegate<T1, T2, TResult>(T1 arg1, T2 arg2);
```

<br/>宣告符合該簽章的函式
```csharp
public string string Output(string name, int age)
{
    return $"Hello World, Name: {name}, Age: {age}";
}
```

<br/>以該委派去達到同樣效果
```csharp
myDelegate<string, int, string> myFuncDel = Output;
Console.WriteLine(myFuncDel("John", 30));
```
>Hello World, Name: John, Age: 30

#### 用 C# 2.0 的 Anonymous Method:
```csharp
myDelegate<string, int, string> myFuncDel = new myDelegate<string, int, string>(delegate(string name, int age){
    return $"Hello World, Name: {name}, Age: {age}";
});
Console.WriteLine(funcDel("John", 30));
```
>Hello World, Name: John, Age: 30

<br>看到這裡, 大概看得出來 Func 就是幫你先把委派定義好, 少了宣告 delegate 的動作
```csharp
Func<string, int, string> myFuncDel = Output;
Console.WriteLine(myFuncDel("John", 30));
```
>Hello World, Name: John, Age: 30

<br/>結合 lambda
```csharp
Func<string, int, string> myFuncDel = (name, age) => $"Hello World, Name: {name}, Age: {age}";;
Console.WriteLine(myFuncDel("John", 30));
```
>Hello World, Name: John, Age: 30

## Aciton 委派
依照它的定義, 是一個泛型委派(generic delegate)，一個有2個參數為泛型型別 T1、T2，且無回傳值的委派
<br/>linq 的 foreach 就是這種作法
```csharp
public delegate void Action()
```

#### 用 C# 1.0 的 delegate:
宣告具有同樣簽章(signature)的委派
```csharp
public delegate void myDelegate();
```

<br/>宣告符合該簽章的函式
```csharp
public string string Output()
{
    Console.WriteLine("Hello World");
}
```

<br/>以該委派去達到同樣效果
```csharp
myDelegate myFuncDel = Output;
myFuncDel();
```
>Hello World

#### 用 C# 2.0 的 Anonymous Method:
```csharp
myDelegate myFuncDel = new myDelegate(delegate(){
    Console.WriteLine("Hello World");
});
funcDel();
```
>Hello World

<br>看到這裡, 大概看得出來 Func 就是幫你先把委派定義好, 少了宣告 delegate 的動作
```csharp
Action myFuncDel = Output;
myFuncDel();
```
>Hello World

<br/>結合 lambda
```csharp
Action myFuncDel = () => "Hello World";
myFuncDel();
```
>Hello World

## Aciton<T1, T2>  委派
依照它的定義, 一個不用任何參數，且無回傳值的委派
<br/>linq 的 foreach 就是這種作法
```csharp
public delegate void Action<in T1, in T2>(T1 arg1, T2 arg2)
```

#### 用 C# 1.0 的 delegate:
宣告具有同樣簽章(signature)的委派
```csharp
public delegate void myDelegate<T1, T2>(T1 arg1, T2 arg2);
```

<br/>宣告符合該簽章的函式
```csharp
public string string Output(string name, int age)
{
    Console.WriteLine($"Hello World, Name: {name}, Age: {age}");
}
```

<br/>以該委派去達到同樣效果
```csharp
myDelegate myFuncDel = Output;
myFuncDel("John", 30);
```
>Hello World, Name: John, Age: 30

#### 用 C# 2.0 的 Anonymous Method:
```csharp
myDelegate<string, int> myFuncDel = new myDelegate(delegate(string name, int age){
    Console.WriteLine($"Hello World, Name: {name}, Age: {age}");
});
funcDel("John", 30);
```
>Hello World, Name: John, Age: 30

<br>看到這裡, 大概看得出來 Func 就是幫你先把委派定義好, 少了宣告 delegate 的動作
```csharp
Action<string, int> myFuncDel = Output;
myFuncDel("John", 30);
```
>Hello World, Name: John, Age: 30

<br/>結合 lambda
```csharp
Action<string, int> myFuncDel = () => Console.WriteLine($"Hello World, Name: {name}, Age: {age}");
myFuncDel("John", 30);
```
>Hello World, Name: John, Age: 30

## Predicate<T>  委派
依照它的定義, 是一個泛型委派(generic delegate)，一個有參數為泛型型別 T，且傳回值為 bool 的委派,
<br/>linq 的 Where 就是這種作法

```csharp
public delegate bool Predicate<in T>(T arg)
```

<br/>如果用 Func<T, bool>也會得到同樣效果

#### 用 C# 1.0 的 delegate:
宣告具有同樣簽章(signature)的委派
```csharp
public delegate bool myDelegate<T>(T arg);
```

<br/>宣告符合該簽章的函式
```csharp
public bool Output(int age)
{
    return age > 10;
}
```

<br/>以該委派去達到同樣效果
```csharp
myDelegate<int> myFuncDel = Output;
Console.WriteLine(myFuncDel(30));
```
>True

#### 用 C# 2.0 的 Anonymous Method:
```csharp
myDelegate<int> myFuncDel = new myDelegate<int>(delegate(int age){
    return age > 10;
});
Console.WriteLine(funcDel(30));
```
>True

<br>看到這裡, 大概看得出來 Func 就是幫你先把委派定義好, 少了宣告 delegate 的動作
```csharp
Predicate<int> myFuncDel = Output;
Console.WriteLine(myFuncDel("John", 30));
```
>True

<br/>結合 lambda
```csharp
Predicate<int> myFuncDel = age => age > 10;
Console.WriteLine(myFuncDel(30));
```
>True