# Design Pattern

## Factory

### 簡單工廠

* 用於取代高耦合的New instance，簡單暴力，適合工廠變動性不大的情境

    ```C#
    // SimpleFactory.cs
    public class SimpleFactory
    {
        public static IService CreateService()
        {
            return new MyService();
        }
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

### ??

* 貌似適合擴充情境使用?

    ```C#
    // ??.cs
    public class WhatIsThisPattern
    {
        protected WhatIsThisPattern(/*...*/)
        {
            //...
        }
    
        public static WhatIsThisPattern Type1(/*...*/)
        {
            return new WhatIsThisPattern(/*...*/);
        }

        public static WhatIsThisPattern Type2(/*...*/)
        {
            return new WhatIsThisPattern(/*...*/);
        }
    }
    ```

    ```C#
    // Program.cs
    public static Main(string[] args)
    {
        WhatIsThisPattern myService = WhatIsThisPattern.Type1();
        //myService...
    }
    ```
