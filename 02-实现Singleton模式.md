# 题目
设计一个类，我们只能生成该类的一个实例。

书上用的C#语言解答本题。

# 不好的解法一：只适用于单线程环境
```c#
public sealed class Singleton1
{
    private Singleton1()
    {
    }

    private static Singleton1 instance = null;
    public static Singleton1 Instance
    {
        get
        {
            if(instance == null)
                instance = new Singleton1();

            return instance;
        }
    }
}
```
要点：
- 构造函数设为私有函数
- 避免重复创建

缺点：
- 只适用于单线程环境

# 不好的解法二：虽然在多线程环境中能工作，但效率不高
```c#
public sealed class Singleton2
{
    private Singleton2()
    {
    }

    private static readonly object syncObj = new object();

    private static Singleton1 instance = null;
    public static Singleton1 Instance
    {
        get
        {
            lock(syncObj)
            {
                if(instance == null)
                    instance = new Singleton2();
            }
            return instance;
        }
    }
}
```
要点：
- 加同步锁保证多线程环境只能得到一个实例

缺点：
- 每次Instance得到实例都会加锁，加锁非常耗时

# 可行的解法：加同步锁前后两次判断实例是否已存在
```c#
public sealed class Singleton3
{
    private Singleton2()
    {
    }

    private static readonly object syncObj = new object();

    private static Singleton1 instance = null;
    public static Singleton1 Instance
    {
        get
        {
            if(instance == null)
            {
                lock(syncObj)
                {
                    if(instance == null)
                        instance = new Singleton3();
                }
            }
            return instance;
        }
    }
}
```
要点：
- instance没创建时才加锁

# 强烈推荐的解法一：利用静态构造函数
```c#
public sealed class Singleton4
{
    private Singleton4()
    {
    }

    private static Singleton4 instance = new Singleton4();
    public static Singleton4 Instance
    {
        get
        {
            return instance;
        }
    }
}
```
要点：
- 静态构造函数可确保只调用一次

缺点：
- 用到Singleton4时实例instance就会被创建，而不是第一次调用Singleton4.Instance时。

# 强烈推荐的解法二：实现按需创建实例
```c#
public sealed class Singleton5
{
    private Singleton5()
    {
    }


    public static Singleton5 Instance
    {
        get
        {
            return Nested.instance;
        }
    }

    class Nested
    {
        static Nested()
        {
        }

        internal static readonly Singleton5 instance = new Singleton5();
    }
}
```
要点：
- 避免了Singleton4解法的缺点。
- 利用私有嵌套类型的特性，真正按需创建实例。

# 完整代码
```c#
using System;

namespace _02_Singleton
{
    public sealed class Singleton1
    {
        private Singleton1()
        {
        }

        private static Singleton1 instance = null;
        public static Singleton1 Instance
        {
            get
            {
                if (instance == null)
                    instance = new Singleton1();

                return instance;
            }
        }
    }

    public sealed class Singleton2
    {
        private Singleton2()
        {
        }

        private static readonly object syncObj = new object();

        private static Singleton2 instance = null;
        public static Singleton2 Instance
        {
            get
            {
                lock (syncObj)
                {
                    if (instance == null)
                        instance = new Singleton2();
                }

                return instance;
            }
        }
    }

    public sealed class Singleton3
    {
        private Singleton3()
        {
        }

        private static object syncObj = new object();

        private static Singleton3 instance = null;
        public static Singleton3 Instance
        {
            get
            {
                if (instance == null)
                {
                    lock (syncObj)
                    {
                        if (instance == null)
                            instance = new Singleton3();
                    }
                }

                return instance;
            }
        }
    }

    public sealed class Singleton4
    {
        private Singleton4()
        {
            Console.WriteLine("An instance of Singleton4 is created.");
        }

        public static void Print()
        {
            Console.WriteLine("Singleton4 Print");
        }

        private static Singleton4 instance = new Singleton4();
        public static Singleton4 Instance
        {
            get
            {
                return instance;
            }
        }
    }

    public sealed class Singleton5
    {
        Singleton5()
        {
            Console.WriteLine("An instance of Singleton5 is created.");
        }

        public static void Print()
        {
            Console.WriteLine("Singleton5 Print");
        }

        public static Singleton5 Instance
        {
            get
            {
                return Nested.instance;
            }
        }

        class Nested
        {
            static Nested()
            {
            }

            internal static readonly Singleton5 instance = new Singleton5();
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // 也会打印An instance of Singleton4 is created.
            Singleton4.Print();

            // 不会打印An instance of Singleton5 is created.
            Singleton5.Print();
        }
    }
}
```
运行结果：
```
An instance of Singleton4 is created.
Singleton4 Print
Singleton5 Print
```