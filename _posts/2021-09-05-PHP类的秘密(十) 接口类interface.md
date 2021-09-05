---
title: "PHP类的秘密(十) 接口类interface"
date: 2021-09-05
---


用这个系列的最后几篇总结一下抽象类, 接口类和trait的详细用法和规则.

## 接口类的作用:

1. 首先接口类可以实现多继承, 这个是其他类不具备的. 所谓多继承就是一个子类可以继承多个父类. 在实际应用中, 常用于多个数据库的服务访问、多个支付网关、不同的缓存策略等.

1. 接口类的意义就像他的名字一样, 将程序进行封装后, 里面是一个黑匣子, 其他程序想要调用该程序, 只需要按照接口类的指示进行传参来实现所需的功能. 定义了接口就像定义了一个协议, 所有对象都按照这个协议进行交互. 

1. 因为实现了同一个接口，所以开发者创建的对象虽然源自不同的类，但可能可以交换使用。

## 定义接口

    interface dfds
    {
        const A=5;
        function use();
        static function dsf();
        function __clone();
        public $a;   //报错: Interfaces may not include member variables
        protected function test();  //报错: Access type for interface method dfds::test() must be omitted
        function fail(){echo 'test'};  //报错: Interface method cannot have body
    }
    
通过上例, 可以看出接口定义的规则:
1. 可以包含常量;

2. 可以包含方法名称;
3. 可以包含静态方法名称;
4. 可以包含魔术方法__;
4. 方法必须是public的, 这是接口的意义决定的.
4. 不能有成员变量;
5. 不能有方法体, 必须由执行子类去定义.

这里要注意一下常量, 如果接口中定义了常量, 则执行子类中就不能定义同名常量. 这一点和继承不太一样:

    interface SDF
    {
        const B = 'Interface constant';
    }
    class dfd implements SDF
    {
        const B='Interface constant';    //报错: Cannot inherit previously-inherited or override constant B from interface SDF
    }
    
## 实现接口类implements
要实现一个接口，使用 implements 操作符。一个子类可以同时继承一个父类, 并执行多个接口: 

    class One                 //定义普通父类
    {
        public $var;
        const B=10;
    }

    interface Usable          //定义接口1
    {
        const A=5;
        function use();
    }

    interface Updatable         //定义接口2
    {
        function test(int $a);
    }

    class Two extends One implements Usable, Updatable  //子类继承父类one, 同时执行接口1和2
    {
        function test(int $a=5)     //参数名命保持统一, 并符合兼容性要求
        {
            echo'test';
        }
        function use()
        {
            echo 'use'; 
        }
    }

通过上例我们可以看到执行子类的一些规则:

1. 当继承和执行同时定义时, 关键词顺序至关重要： 'extends' 必须在前面.

2. 执行子类必须定义所有接口类的方法体, 否则会报错.

3. 执行多个接口时用逗号分隔. 
4. 如果接口方法含有指定参数, 则执行子类的成员方法必须符合参数定义兼容性要求, 比如不能将父类的可选参数改为必填参数, 不能删除父类参数等.


**如果两个接口中有同名方法怎么办呢?**

    interface oii
    {
        function jkl();
    }
    interface poi
    {
        function jkl();    //定义同名方法
    }

    class fds implements oii, poi
    {
        function jkl()         //只定义一次就好, 定义两次会报错;
        {
            echo'same';
        }
    }
如果有同名方法, 只需要在执行子类中, 定义一次该方法就可以.

**可以用抽象类来执行接口么**


    interface A
    {
        public function foo(string $s): string;
        public function bar(int $i): int;
    }

    abstract class B implements A          // 抽象类可能仅实现了接口的一部分。
    {
        pubic function foo(string $s): string
        {
            return $s . PHP_EOL;
        }
    }

    class C extends B                    // 扩展该抽象类时必须实现剩余部分。
    {
        public function bar(int $i): int
        {
            return $i * 2;
        }
    }
    
抽象类可以用来执行接口, 而且不需要实现所有接口方法. 普通类可以继续继承抽象类, 并实现剩余接口方法

## 接口类继承--子接口
接口也可以通过 extends 操作符扩展子接口


    interface A
    {
        public function foo();
    }

    interface B
    {
        public function bar();
    }

    interface C extends A, B     //子接口C同时继承了A,B接口;
    {
        public function baz();
    }

    class D implements C        //需要实现所有父类接口的方法
    {
        public function foo()
        {
        }

        public function bar()
        {
        }

        public function baz()
        {
        }
    }
子接口可以同时继承多个父类接口, 而且不需要实现父接口的方法, 可以定义自己的方法, 而子接口的执行子类需要实现所有父类接口的方法. 


感谢阅读, 如有不准确或错误之处, 请留言指正, 我会及时修正, 感谢!

欢迎喜欢技术的同学和我交流, 微信1296386616

总结不易, 请勿私自转载, 否则别怪老大爷不客气!

参考资料:

www.php.net PHP官方文档
