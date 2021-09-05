---
title: "PHP类的秘密(六) static, $this, self, parent和后期静态绑定"
date: 2021-09-05
---




全网真正把这几个代词讲清楚的很少. 而他们又非常重要, 用不好就会掉进坑里. 我今天就来总结一下PHP类中这些代词的用法和区别, 以及当中的坑.

先从最复杂的static说起.

## static

static有三种用法:

1. 表示静态, 我们在静态那篇专门介绍过, 在此不赘述, 总结一下就是用在声明属性和声明成员方法之前, 可以达到不需要实例化也能直接调用的目的, 并且让静态属性和类进行绑定.

1. 就是代词. 可以用在成员方法的表达式中, 指代调用它的类;

1. 成员方法返回值类型, 用在成员方法声明中, 限制返回值必须是调用类的对象;

我们来举例看一下:

    class Product {               
        public static function getNew( ): static {     
            $new = new static;   
            return $new;
        } 
    }
    class SubProduct extends Product{
    }
    
    $p1 = Product::getNew( );
    $p2 = SubProduct::getNew();
    var_dump($p1);    //object(Product)#24
    var_dump($p2);    //object(SubProduct)#27

这个例子用到了static的后两个用法:
1. 数据返回类型static, 限制了返回值必须是调用类的对象. 所以在这里父类Product调用的, 那么返回的也必须是Product的对象;

1. \$new= new static; 这里的static指代的就是调用它的类名. 所以SubProduct::, static就指代SubProduct, 这句话可以翻译成$new= new SubProduct, 一个对象创建语句.

只要记住一点: **static用在方法体内, 谁调用它, 它就指代谁的类.**

当这个"它", 是类的时候, 比如上面的例子, 就是静态调用; 当它是实例化的对象的时候, 就是非静态调用.

很简单是不是? 先别着急自我肯定, 我留个坑, 先讲其他几个代词, 最后再回来看这个"**静态调用**".

## $this

$this是一个到当前对象的引用. 简单说, 就是谁调用我, 我就是谁. 

同时$this不能用来访问静态属性, 因为静态属性是和类绑定的, 只能由static, self和parent访问;

我们举例来看一下$this的含义:

    class MyClass1
    {
        public $public = 'Public';
        protected $protected = 'Protected';
        private $private = 'Private';

        function printHello()
        {
            echo $this->public;
            echo $this->protected;
            echo $this->private;
        }
    }
    
    class MyClass2 extends MyClass1
    {
        public $public = 'Public2';
        protected $protected = 'Protected2';
        private $private = 'Private2';
    }
    
    $obj = new MyClass1();
    $obj -> printHello();   //Public Protected Private
    $obj2 = new MyClass2();
    $obj2 -> printHello();  //Public2 Protected2 Private
    
当\$obj2调用父类方法时, 前两个都能输出子类自己的public和protected属性, 所以\$this在这里指代的就是引用它的对象\$obj2, \$this->public等价于$obj2->public:

注意到:$obj2->printHello()最后却输出了父类的private属性, 这个就是我们在本系列第三篇<继承>时, 讲到的私有属性的就近原则: 

当对象含有两个同名属性的时候, 通过哪个类调用, 就返回那个类的属性.

### 就近原则

$this的就近原则不仅适用于private属性, 同样适用于private成员方法. 而static, self, parent都没有这个特征. 我们举例看一下:

    class AA 
    {     
        private function foo() {
            echo "success!\n";
        }
        
        public function test() {
            $this->foo();
            static::foo();
        }
    }

    class BBBB extends AA 
    {
    }

    class CCC extends AA {
        private function foo() 
        {
            echo 'CCC';
        }
    }

    $b = new BBBB();
    $b->test();       //Success Success
    $c = new CCC();
    $c->test();       //Success  error:Call to private method CCC::foo() from scope AA

先说BBBB子类, 它完全继承了AA父类的方法. 所以此时\$this->foo()可以等价于$b->foo(), 而static::foo()等价于A::foo(), 输出两个success;

再看CCC这个子类, 他其实继承了AA父类的两个方法, **其中private function foo()是隐形继承**. 但同时, 它也有自己的private function foo(), 那么它同时有两个同名的方法. 这个时候$this和static指代谁就有了不同的结果.

上一节说过static指代调用的类, 这里$c属于CCC, 那么static::foo()就等价于CCC::foo, 但是CCC::foo是私有属性, 只能在类内访问, 而test()确实属于AA父类, 所以导致报错.

**那么为什么\$this->foo()会输出success呢?** 我们提到$this针对private属性和方法有就近原则, 就是在哪个类调用, 就指代那个类.

在上例中public function test()是属于AA父类的, 所以$this->foo(), 这里指代的就是父类的private function foo(), 所以会输出"success". 如果把上例中的test()放到子类, 那么输出的结果就是'CCC', 大家可以在本地验证一下.

**总结一下: $this指代被调用的对象, 但在处理private属性和方法时, 会遵循就近原则, 会指代所属方法所在的类**




## self & parent

这两个我放在一起讲, 是因为他们有共性: 都是跟所在成员方法的类绑定. parent指代的是所在成员方法类的父类.

通过下面这个例子, 我们看一下self, parent的含义, 以及和static的区别:

    class fooo                //定义父类及成员方法
    {
        public function something()
        {
            echo __CLASS__; // 返回当前方法所在类名
            var_dump($this);
        }
    }

    class fooo_bar extends fooo        //定义子类并重写父类方法
    {
        public function something()
        {
            echo __CLASS__; // fooo_bar
            var_dump($this);
        }
        
        public function call()       //定义方法调用其他类的方法
        {
            echo self::something();   // self
            echo parent::something(); // parent
            echo static::something(); // current
        }
    }

    class fooo_bar_baz extends fooo_bar    //定义子类的子类, 并重写方法
    {
         public function something()
         {
            echo __CLASS__; // fooo_bar_baz
            var_dump($this);
          }
    }

    $obj = new fooo_bar_baz();
    $obj->call();  
    
    //**output**:
    fooo_bar
    fooo
    fooo_bar_baz

好的, 上面这个例子, 我构建了一个父,子, 孙三个类, 并在子类创建了成员方法去调用3个类的成员方法, 然后在孙类创建了对象去调用子类的方法, 得到的结果:
1. self指代的是方法所在的类, 和谁调用的无关, 所以在上例中就是fooo_bar;

2. parent指代当前方法所在类的父类, 和谁调用的无关, 所以即便是孙类调用的, 但指向的仍然是子类的父类fooo;

3. static, 就如我们第一节讲到的, 和调用的类绑定, 这里$obj是属于孙类的, 所以输出的就是孙类fooo_bar_baz;

4. `__CLASS__`是PHP中的一个魔法变量, 和get_class()函数的作用一样, 都是返回调用对象的类名;


## static后期静态绑定的坑
看到这里, 你是不是感觉对这几个代词已经成竹在胸了呢? 先别着急自我肯定. 我们来看看最开始留的那个坑: 静态调用.

先解释一下什么是静态调用, 就是通过双引号::对静态变量和静态方法的直接访问.

非静态调用就是由实例化的对象通过->进行调用.

当static遇上静态调用的时候, 神奇的事情就发生了:

    class A {
        public static function foo() {
            static::who();
        }

        public static function who() {
            echo __CLASS__."\n";
        }
    }

    class B extends A {
        public static function test() {
            A::foo();
            parent::foo();
            self::foo();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
    }

    class C extends B {
        public static function who() {
            echo __CLASS__."\n";
        }
    }

    C::test(); //结果: A C C

怎么样? 有没有被上面的结果打脸? 我第一次以为是AU AU BU, 结果脸都被打蒙了. 

这是因为static在静态调用时会有一些不一样:

**在静态调用的情况下, 不管中间经过哪些代词, 它都指向明确指定的那个类.**  这就是所谓的"后期静态绑定", static此时指向的是在上一个“非转发调用”（non-forwarding call）的类. 

换成人话就是: 所谓"非转发调用", 就是指明确指定的那个类. 就是上例中的C::和A::, 不需要进行转发, 直接明确了是那个类在调用.

那么"转发调用", 就是指self::，parent::，static::, 这样的代词, 没有明确指定类名, 只是做类名的forward.

好了, 现在让我们再来看一下上面的例子:

1. A, 结果很直接, 因为test()里面明确了A:FOO(), 所以foo()中的static指代明确指定的A, 调用A:WHO方法;

2. 后两个语句parent:foo()和 self:foo(), 实际上都是C通过B的方法test(), 调用了A的方法foo(), 那么此时static到底指代谁?

3. 结合static静态调用的特性, static指向明确指定的那个类, 而此时self和parent都只是转发C::, 所以static此时指代的都是C::, 所以会输出两个C

**这个特性, 只有static关键词才有, 可以简单理解为"在静态调用时, 明确指定的那个类名"即可. **



我汇总了一张表, 更清晰地展示了这4个代词指代的对象和用法上的区别:

![1](https://user-images.githubusercontent.com/76157254/132111982-52a98adb-4247-48c2-b099-698fa5de24b2.jpg)

感谢阅读, 抛砖引玉, 有不准确和错误的地方请各位大神批评指正, 我都会及时回复, 拜谢!

欢迎学习编程的小伙伴儿和我交流, 共同学习, 一起成长!


总结不易, 请勿私自转载, 否则别怪老大爷不客气!

参考资料:

www.php.net PHP官方文档
