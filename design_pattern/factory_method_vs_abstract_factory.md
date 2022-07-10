# Factory Method VS Abstract Factory

- [Factory Method VS Abstract Factory](#factory-method-vs-abstract-factory)
  - [兩者差別](#兩者差別)
  - [從 Template Method 到 Factory Method/Abstract Factory](#從-template-method-到-factory-methodabstract-factory)
    - [改成 Factory Method](#改成-factory-method)
    - [改成 Abstract Factory](#改成-abstract-factory)

---
## 兩者差別

Factory Method
+ 工廠方法只用來生成一個產品
+ 工廠方法的工廠是方法
+ 工廠方法生成的產品，可以給客戶端或自己使用

<br/>Abstract Factory
+ 抽象工廠只是一個產品群，且互相有關係
+ 抽象工廠的工廠是類別
+ 抽象工廠生成的產品，只給客戶端，自己不使用
+ 缺點：每加入一個新產品，就須改變 interface，然後所有繼承該 interface 的子類別都須跟著更動

---
## 從 Template Method 到 Factory Method/Abstract Factory

<br/>如下程式碼，若想改函式內容就繼承 MvcEngine 去覆寫函式
```csharp
public class MvcEngine
{
    public void Start(Uri address)
    {
        while(true)
        {
            Request request = this.Listen(address);

            Task.Run(() => {
                Controller controller = this.ActivateController(request);
                View view = this.ExecuteController(controller);
                this.RenderView(view);
            });
        }
    }

    protected virtual Request Listen(Uri address)
    {
        return new Request();
    }

    protected virtual Controller ActivateController(Request request)
    {
        return new Controller();
    }

    protected virtual View ExecuteController(Controller controller)
    {
        return new View();
    }

    protected virtual void RenderView(View view)
    {
        return;
    }
}
```

<br/>

### 改成 Factory Method
一樣是繼承 MvcEngine，去覆寫函式去傳回想使用的實體類別
```csharp
public class MvcEngine
{
    public void Start(Uri address)
    {
        while(true)
        {
            Request request = this.GetListener().Listen(address);

            Task.Run(() => {
                Controller controller = this.GetControllerActviator().ActivateController(request);
                View view = this.GetControllerExecutor().ExecuteController(controller);
                this.GetViewRender().RenderView(view);
            });
        }
    }

    protected virtual Listener GetListener()
    {
        return new Listener();
    }

    protected virtual ControllerActviator GetControllerActviator(Request request)
    {
        return new ControllerActviator();
    }

    protected virtual ControllerExecutor GetControllerExecutor(Controller controller)
    {
        return new ControllerExecutor();
    }

    protected virtual ViewRender GetViewRender(View view)
    {
        return new ViewRender();
    }
}
```

<br/>

### 改成 Abstract Factory
生成實體類別的控制權移到 EngineFactory，繼承 EngineFactory，繼承去控制傳回想使用的類別
```csharp
public class MvcEngine
{
    public EngineFactory Factory { get; private set; 
    }
    public MvcEngine(EngineFactory factory == null)
    {
        this.Factory = factory ?? new EngineFactory();
    }

    public void Start(Uri address)
    {
        while(true)
        {
            Request request = this.Factory.GetListener().Listen(address);

            Task.Run(() => {
                Controller controller = this.Factory.GetControllerActviator().ActivateController(request);
                View view = this.Factory.GetControllerExecutor().ExecuteController(controller);
                this.Factory.GetViewRender().RenderView(view);
            });
        }
    }
}
```

<br/>EngineFactory 類別
```csharp
public class EngineFactory
{
    protected virtual Listener GetListener()
    {
        return new Listener();
    }

    protected virtual ControllerActviator GetControllerActviator(Request request)
    {
        return new ControllerActviator();
    }

    protected virtual ControllerExecutor GetControllerExecutor(Controller controller)
    {
        return new ControllerExecutor();
    }

    protected virtual ViewRender GetViewRender(View view)
    {
        return new ViewRender();
    }
}
```