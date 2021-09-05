---
title: "PHP类的秘密(11) 抽象类abstract"
date: 2021-09-05
---


## 为什么要有抽象类?
**类已经很抽象了, 为啥还要弄个抽象类abstract呢? **

因为抽象类的意义在于, 它规定了一个事物的基本属性和行为类型, 然后让其子类根据它定的规范去具体实现方法. 就好像, 虽然都属于车这个抽象类, 但是具体有卡车, 越野车, 摩托车, 装甲车等等. 这些子类扩展了车, 也让车这个类更加灵活.

相当于是在类的基础上, 又抽象了一层, 但这层抽象还具有了一个通用模板的意义.

## 什么是抽象类abstract

想要了解抽象类, 我们先来看看它与普通类的区别:

1. 必须由关键词abstract定义.

1. 首先PHP规定其不能实例化. 因为太抽象了, 实例化没有意义. 

1. 抽象类可以不含有抽象方法, 但含有抽象方法的类, 必须声明为抽象类. 

1. 抽象方法只定义方法名称和参数, 不定义具体方法实现的类. 具体的方法实现,由其子类去定义. 这一点和接口类是一样的.

1. 抽象方法不能是私有的, 只能是public 或 protected, 这是抽象方法的性质决定的.

我们看一个例子:

    abstract class AbstractClass
    {
        static $c=10;
        
        // 强制要求子类定义抽象方法
        abstract protected function getValue();
        abstract public function prefixValue($prefix);  

        // 普通方法（非抽象方法）
        public function printOut() {
            print $this->getValue() . "\n";
        }
        
        static function staticone(){
            echo "test\n";
        }
     }

    class ConcreteClass1 extends AbstractClass
    {
        protected function getValue() {
            return "ConcreteClass1";
        }

        public function prefixValue($prefix) {
            return "{$prefix}ConcreteClass1";
        }
    }

    $class1 = new ConcreteClass1;
    $class1->printOut();  //ConcreteClass1
    echo $class1->prefixValue('FOO_');  //FOO_ConcreteClass1
    echo AbstractClass::$c;          //10
    AbstractClass::staticone();      //'test'

通过上例可以看出:
- 抽象方法没有方法体, 只是声明了其调用方式（参数），不能定义其具体的功能实现。功能实现只能在子类中完成, 继承一个抽象类的时候，子类必须定义父类中的所有抽象方法

- 抽象类中可以含有普通方法

- 抽象类可以定义静态属性和静态方法, 并可以在类外直接调用, 这个和普通类是一样.



## 我们来说一下抽象类的可见度:

    abstract class Adapter
    {
        protected $name;
        private $priv;
        abstract public function getName(): string;
        abstract protected function setName(string $value);
    }

    class AdapterFoo extends Adapter
    {
        public function getName($d=5): string
        {
            return $this->name;
        }
        public function setName(string $value,$a=0): self     //子类继承抽象类,可以增加返回类型限制, 可以增加可选参数;
        {
            $this->name = $value;
            return $this;
        }
    }
通过上例可以看出: 

- **在子类中的方法, 访问控制必须和抽象类中一样, 或者更为宽松**。例如某个抽象方法被声明为受保护的，那么子类中实现的方法就应该声明为受保护的或者公有的，而不能定义为私有的。

- 同时, 抽象类的子类成员方法, 必须要兼容父类的成员方法参数规则, 包括: 由返回类型的, 必须保持, 无返回类型的可以添加返回类型; 父类必选子类可以是可选, 父类无参数子类可以是可选等

- 父类可以定义私有属性, 同样会被子类继承.

- 总体来说, 抽象类的继承规则, 和普通类是一致的.

**抽象类也可以是子类**, 可以用抽象类来extends另一个抽象类, 抽象子类不需要实现父类的抽象方法, 可以只定义自己的抽象方法, 而它的子类必须实现所有抽象方法: 

    abstract class class1 {
        abstract public function someFunc();
    }
    abstract class class2 extends class1 {
        abstract protected function anyFunc();
    }    //用一个抽象类,继承一个抽象类, 不需要实现抽象方法
    class Zhuozi extends class2
    {                               //子类需要定义所有父类的抽象方法
        function someFunc()
        {
            echo "test";
        }

        function anyFunc()
        {
            echo 'any';
        }
    }

## 最后, 我们看看抽象类和接口类的区别和联系:

    interface AEE
    {
        public function foo(string $s): string;

        public function bar(int $i): int;
    }

    abstract class BEE implements AEE
    {
        public function foo(string $s): string
        {
        return $s . PHP_EOL;
    }
    }// 抽象类可能仅实现了接口的一部分。

    class CEE extends BEE
    {
        public function bar(int $i): int
        {
        return $i * 2;
        }
    }// 扩展该抽象类时必须实现剩余部分。
    
我们会用抽象类来实现一部分接口的成员方法, 当这样的情况发生时, 抽象类就作为其他类的基类, 并且拥有一个内置的接口.  

和接口类比起来, 抽象类定义了一部分公共方法, 供子类继承, 而接口类只定义了方法名称和参数;

抽象类不能多继承.


感谢阅读, 如有不准确或错误请留言, 我会及时更正, 感谢!

欢迎喜欢技术的同学和我交流, 微信1296386616

总结不易, 请勿私自转载, 否则别怪老大爷不客气!

参考资料:
WWW.PHP.NET PHP官方文档
