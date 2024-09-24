# Json Deserialize Interface

[參考來源](https://gist.github.com/tonysneed/5e7988516b081d454cde95b5d729e1af)

使用平台: .Net Core 7.0

使用套件:
- System.Text.Json

定義以下的類別：

```csharp
public interface IMyClass
{
    string Name { get; set; }
}

public class MyClass : IMyClass
{
    public string Name { get; set; }
}
```

我們可以序列化 `IMyClass` 介面，因為它實際上是 `MyClass` 類別，但是無法反序列化為 `IMyClass` 介面，因為介面無法被實際化。  

```csharp
var myClass = new MyClass { Name = "Child" };
var json = JsonSerializer.Serialize(myClass);
var parent2 = JsonSerializer.Deserialize<IMyClass>(json);
```
>System.NotSupportedException: Deserialization of interface types is not supported. Type 'IMyClass'.

我們需要告訴 `JsonSerializer.Deserializer` 在遇到介面時換成哪個類別，以下是解法：  
Create a JsonConverter for the interface.  

```csharp
public class InterfaceConverter<TImplementation, TInterface> : JsonConverter<TInterface>
    where TImplementation : class, TInterface
{
    public override TInterface? Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        => JsonSerializer.Deserialize<TImplementation>(ref reader, options);

    public override void Write(Utf8JsonWriter writer, TInterface value, JsonSerializerOptions options)
    {
    }
}
```

Create a generic JsonConverterFactory to create the interface converter.

```csharp
public class InterfaceConverterFactory<TImplementation, TInterface> : JsonConverterFactory
{
    private Type ImplementationType { get; }
    private Type InterfaceType { get; }

    public InterfaceConverterFactory()
    {
        ImplementationType = typeof(TImplementation);
        InterfaceType = typeof(TInterface);
    }

    public override bool CanConvert(Type typeToConvert)
        => typeToConvert == InterfaceType;

    public override JsonConverter? CreateConverter(Type typeToConvert, JsonSerializerOptions options)
    {
        var converterType = typeof(InterfaceConverter<,>).MakeGenericType(ImplementationType, InterfaceType);
        return Activator.CreateInstance(converterType) as JsonConverter;
    }
}
```

在使用時，需要將 `InterfaceConverterFactory` 加入到 `JsonSerializerOptions` 的 `Converters` 屬性中。  

```csharp
var deserializerOptions = new JsonSerializerOptions
{
    Converters = {
        new InterfaceConverterFactory<MyClass, IMyInterface>(),
        new InterfaceConverterFactory<MyClass2, IMyInterface2>()
    }
};

// Deserialize to the interface.
// This will trigger an exception without the options.
var myInterface = JsonSerializer.Deserialize<IMyInterface>(json, deserializerOptions);
if (myInterface is MyClass myClass2)
{
    // Use Implementation
}
```
