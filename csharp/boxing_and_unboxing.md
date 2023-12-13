# [C#]Boxing & Unboxing

## Boxing

將實質型別指派給參考型別。  

Boxing is used to store value types in the garbage-collected heap. Boxing is an implicit conversion of a value type to the type object or to any interface type implemented by this value type. Boxing a value type allocates an object instance on the heap and copies the value into the new object.

## Unboxing

將參考型別指派給實質型別。  

Unboxing is an explicit conversion from the type object to a value type or from an interface type to a value type that implements the interface. An unboxing operation consists of:
- Checking the object instance to make sure that it is a boxed value of the given value type.
- Copying the value from the instance into the value-type variable.

```csharp
int i = 123;      // a value type 
object o = i;     // boxing 
int j = (int)o;   // unboxing
```

![Unboxing Conversion Operation](imgs/unboxing_conversion_operation.gif)

優點
- 易操作，所有的東西都可以塞 HashTable、ArrayList
缺點
- 不易除錯，型別錯誤在 runtime 才會知道
- 轉換很耗效能