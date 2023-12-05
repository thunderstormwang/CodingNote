# [C# 2.0]Static Constructor

A static constructor is used to initialize any static data, or to perform a particular action that needs performed once only. It is called automatically before the first instance is created or any static members are referenced.

Static constructors have the following properties:
- A static constructor does not take access modifiers or have parameters.
- A static constructor is called automatically to initialize the class before the first instance is created or any static members are referenced.
- A static constructor cannot be called directly.
- The user has no control on when the static constructor is executed in the program.
- A typical use of static constructors is when the class is using a log file and the constructor is used to write entries to this file.
- Static constructors are also useful when creating wrapper classes for unmanaged code, when the constructor can call the LoadLibrary method.

```csharp
public class Bus
{
    static Bus()
    {
        Console.WriteLine("Bus static constructor invoked");
    }
    
    public Bus()
    {
        Console.WriteLine("Bus constructor invoked");
    }

    public static void Drive()
    {
        Console.WriteLine("The Drive method invoked");
    }
}
```

如果這樣呼叫，static contructor 會先被呼叫
```csharp
Bus.Drive();
```
>Bus static constructor invoked  
The Drive method invoked

如果這樣呼叫，static contructor 會先被呼叫
```csharp
var bus = new Bus();
```
>Bus static constructor invoked  
Bus constructor invoked
