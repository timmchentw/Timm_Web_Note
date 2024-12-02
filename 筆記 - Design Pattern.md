# Design Pattern

- [Design Pattern](#design-pattern)
  - [Factory](#factory)
    - [相依性注入](#相依性注入)
    - [簡單工廠](#簡單工廠)
    - [抽象工廠](#抽象工廠)
  - [策略模式](#策略模式)
  - [Template 範本模式](#template-範本模式)
  - [Singleton 單例模式](#singleton-單例模式)
  - [Prototype 原型模式](#prototype-原型模式)


## Factory

### 相依性注入

* 將相依方法、參數以Constructor注入其中，Class內不負責相依項目的實例化
* 搭配Interface會降低更多的耦合度
* 優點: 低耦合，符合LSP，且減化掉實例化的邏輯
* 缺點: 程式較不直觀，且需要注意注入的Class的實例狀況 (如Singleton instance需要注意各個引用的class的共用狀況)

    ```C#
    public class DIClass
    {
        private readonly AppSettings _appSettings;
        private readonly IService1 _serivce1;
        private readonly IService2 _service2;

        public DIClass(AppSettings appSettings, IService1 serivce1, IService2 service2)
        {
            _appSettings = appSettings;
            _service1 = service1;
            _service2 = service2;
        }

        public void Method()
        {
            _service1.GetData();
        }
    }
    ```

### 簡單工廠

* 用於取代高耦合的New instance，適合工廠變動性不大的情境
* 可搭配Enum建立不同的實體，符合LSP
* 優點: 簡單暴力，可避免高階方法直接依賴低階方法
* 缺點: 靜態方法尤其限制性，當參數多、需要抽換工廠時較不夠用

    ```C#
    // SimpleFactory.cs
    public class SimpleFactory
    {
        public static IService CreateService()
        {
            return new MyService();
        }

        public static IService CreateService(ServiceType type)
        {
            switch (type)
            {
                case ServiceType.Service1:
                    return new MyService1();
                case ServiceType.Service2:
                    return new MyService2();
            }
        }
    }

    public enum ServiceType
    {
        Service1,
        Service2,
    }
    ```

    ```C#
    // Program.cs
    public static Main(string[] args)
    {
        IService myService = SimpleFactory.CreateService();
        //myService...
    }
    ```

### 抽象工廠

* 適合需擴增工廠的情境、或工廠有需要input parameters的情況

    ```C#
    // AbstractFactory.cs
    public class AbstractFactory : IAbstractFactory
    {
        public AbstractFactory(/*...*/)
        {
            //...
        }
    
        public static IService CreateService()
        {
            return new MyService(/*...*/);
        }
    }
    ```

    ```C#
    // Program.cs
    public static Main(string[] args)
    {
        IAbstractFactory abstractFactory = new AbstractFactory(/*...*/);  // 方便示意，可以用DI時建議替換掉寫法
        IService myService = abstractFactory.CreateService();
        //myService...
    }
    ```

## 策略模式

* 適合常拓展的方法，撰寫新的Class繼承特定Strategy介面，使用時進行抽換
* 優點: 好維護、易抽換、符合SRP
* 缺點: 會產生很多的Class

    ```C#
    public interface IStrategy
    {
        string GetData();
    }

    public class StrategyApi : IStrategy
    {
        public string GetData()
        {

        }
    }

    public class StrategyDb : IStrategy
    {
        public string GetData();
        {

        }
    }

    ```

    ```C#
    static void Main(string[] args)
    {
        IStrategy str1 = new StrategyApi();
        IStrategy str2 = new StrategyDb();

        if (args.Length > 0)
        {
            str1.GetData();
        }
        else
        {
            str2.GetData();
        }
    }
    ```

* 適合搭配工廠模式抽換

    ```C#
    public class SimpleFactory
    {
        public static IStrategy CreateService(ServiceType type)
        {
            switch (type)
            {
                case ServiceType.Service1:
                    return new StrategyApi();
                case ServiceType.Service2:
                    return new StrategyDb();
            }
        }
    }
    ```

    ```C#
    static void Main(string[] args)
    {
        if (args.Length > 0)
        {
            SimpleFactory.CreateService(ServiceType.Service1).GetData();
        }
        else
        {
            SimpleFactory.CreateService(ServiceType.Service2).GetData();
        }
    }
    ```

## Template 範本模式

* 以Abstract class進行實作的模式
* 優點1: 可共用一些方法，減少重複的程式碼
* 優點2: 可在Abstract class定義好固定流程，再由繼承class實做特定細節流程，求同存異
* 缺點1: 對實體Class進行高耦合繼承，須留意相依性，尤其需要**非常注意盡量避免override virtual method**，否則繼承階層一多起來，底層Class改動virtual方法將影響整個繼承鍊(極高風險產生難解的BUG)
* 缺點2: Constructor會互相影響，底層Class新增Parameter時會需要修改所有繼承的Class
* 建議1: 少用Virtual，盡量只使用Abstract method進行Override，避免繼承鍊的破壞
* 建議2: 明確區分private & protected，不必要給繼承class使用的method盡量使用private

## Singleton 單例模式

* 只new一個實體，日後直接共用該實體
* 優點1: 資源效率最大化，可節省記憶體
* 優點2: 降低耦合度，只在特定地方new instance
* 缺點1: 需特別注意共用的情況，由其該class有property時，多個reference改變其值會有不可預測的風險
* 缺點2: 如共用db connection，須注意transaction lock的狀況
  
    ```C#
    public class SingletonClass
    {
        private readonly _myService = new MyService();

        public MyService GetService()
        {
            return _myService;
        }
    }
    ```

### Adapter 適配器模式

* 優點: 在不改變現有Class的情況下，製造出符合Interface要求的內容

    ```C#
    // 程式端需求的介面
    interface ITarget
    {
        public void Query();
    }

    // 轉換前的物件
    class OriginalClass
    {
        public void SpecificQuery()
        {
            System.Console.WriteLine("run specificRequest in Adaptee class");
        }
    }

    // 轉換器模式 → 符合介面
    class ObjectAdapter : ITarget
    {
        private OriginalClass _adaptee;

        public ObjectAdapter(OriginalClass adaptee)
        {
            // 注入原本的類別
            _adaptee = adaptee;
        }
        public void Query()
        {
            _adaptee.SpecificQuery();
        }
    }

    // 實際跑起來的樣子
    public class Program
    {
        static void Main(string[] args)
        {
            OriginalClass adaptee = new OriginalClass();
            ITarget target = new ObjectAdapter(adaptee);
            target.Query();
        }
    }
    ```

## Prototype 原型模式

* 待確認?

    ```C#
    // PrototypeClass.cs
    public class PrototypeClass
    {
        protected PrototypeClass()
        {
            public string Type;
        }
    
        public static PrototypeClass Type1()
        {
            return new PrototypeClass("One");
        }

        public static PrototypeClass Type2()
        {
            return new PrototypeClass("Two");
        }
    }
    ```

    ```C#
    // Program.cs
    public static Main(string[] args)
    {
        PrototypeClass typeClass = PrototypeClass.Type1();
        Console.WriteLine(typeClass.Type);
    }
    ```
