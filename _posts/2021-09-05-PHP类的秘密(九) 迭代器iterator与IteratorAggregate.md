---
title: "PHP类的秘密(九) 迭代器iterator与IteratorAggregate"
date: 2021-09-05
---


这章的内容本来是想和上一篇<对象的秘密>一起讲, 不过碍于篇幅太长, 还是单独开一篇专门总结一下.

## foreach
foreach是遍历对象的工具, 它可以自己单独使用, 也可以配合迭代器, 生成器一同使用. 

下面是foreach单独使用时遍历对象的例子. 非常简单, 就和遍历数组是一样的, 可以用在类内, 也可以用在类外:

    class MyClass
    {
        public $var1 = 'value 1';
        protected $protected = 'protected var';
        private   $private   = 'private var';

        function iterateVisible() 
        {
           foreach($this as $key => $value) {  //类内, foreach定义在成员方法;
           print "$key => $value\n";
           }
        }
    }

    $class = new MyClass();
    foreach($class as $key => $value) {    //作为类外函数读取对象
    print "$key => $value\n";
    }                                  //var1 => value 1, 仅输出public属性
    $class->iterateVisible()       //var1 => value 1, protected => protected var, private => private var 输出所有对象属性;
    
**通过上例, foreach如果定义在类内, 则可以输出所有属性, 如果在类外作为函数调用, 则只可以输出public属性.**

如果对象属性中有数组型, 也可以将上例优化为foreach(\$this->array as \$key=>$value)进行遍历. 

单独使用foreach有其局限, 就是无法让对象自行决定遍历的起点, 以及每次遍历时那些值可用。想让foreach遍历得更加灵活, 就需要迭代器上场了.


## 迭代器iterator{}

迭代器iterator{}的作用是, **当执行foreach遍历对象时, 被自动调用, 规定了遍历的方法**. 

Iterator是一个PHP预定义的接口类, 可以直接使用, 这是它内部的结构:

    Iterator extends Traversable 
    {
    abstract public current(): mixed — 返回当前元素
    abstract public key(): scalar — 返回当前元素的键
    abstract public next(): void — 向前移动到下一个元素
    abstract public rewind(): void — 返回到迭代器的第一个元素
    abstract public valid(): bool — 检查当前位置是否有效
    }
    
可以看到iterator{}是Traversable{}的子类, Traversable{}是检测一个类是否可以使用 foreach 进行遍历的接口。但无法被单独实现, 一般的用法如下:

    if( !is_array( $items ) && !$items instanceof Traversable )
        //Throw exception here

Traversable{}必须由 IteratorAggregate 或 Iterator 接口实现。而使用iterator{}需要定义类实现接口, 如`class MyIterator implements Iterator{}`

iterator迭代器内部包含5种抽象方法, 都需要在其他类中实现. **而且每种方法必须是public的**, 否则无法由foreach, while等函数在类外进行自动调用.

我们来实际看一个例子, 看看foreach是如何调用这些成员方法实现遍历的:

    class myIterator implements Iterator {
        private $position = 0;
        private $array = array
        (
            "firstelement",
            "secondelement",
            "lastelement",
        );  

    public function __construct() {  //构造函数初始化属性
        $this->position = 1;
    }

    function rewind() {              //返回到迭代器的第一个元素, 决定迭代起点
        var_dump(__METHOD__);
        $this->position = 1;
    }

    function current() {                //返回当前属性值
        var_dump(__METHOD__);
        return $this->array[$this->position];
    }

    function key() {                   //返回当前的$key
        var_dump(__METHOD__);
        return $this->position;
    }

    function next() {                   //确定下一次迭代时的位置
        var_dump(__METHOD__);
        ++$this->position;
    }

    function valid() {               //验证是否有值
        var_dump(__METHOD__);
        return array_key_exists($this->array, $this->position);
    }
    }

    $it = new myIterator;

    foreach($it as $key => $value) {     //外部函数调用内部接口
    var_dump($key, $value);
    echo "\n";
    }
    //output:
    string(18) "miIterator::rewind"
    string(17) "miIterator::valid"
    string(19) "miIterator::current"
    string(15) "miIterator::key"
    int(1)
    string(13) "secondelement"

    string(16) "miIterator::next"
    string(17) "miIterator::valid"
    string(19) "miIterator::current"
    string(15) "miIterator::key"
    int(2)
    string(11) "lastelement"

    string(16) "miIterator::next"
    string(17) "miIterator::valid"

    
上例中foreach函数会在遍历时输出了对象\$it的私有属性$array, 并且不是全部输出, 而是从第[1]个元素开始输出, 我们一条条分析以下它是如何做到的:

    string(18) "myIterator::rewind"      //迭代开始前调用rewind方法, 返回到迭代器的第一个元素, 同时执行$this->position = 1;

    string(17) "myIterator::valid"      //验证当前元素$this->array[1]是否有效, 有效则继续执行;

    string(19) "myIterator::current"    // 返回当前元素对应的值$this->array[0]='firstelement'

    string(15) "myIterator::key"       // 返回当前元素的键$this->position = 1

    int(1)                             //var_dump($key)

    string(12) "secondelement"         //var_dump($value)
    
    string(16) "myIterator::next"     //向前移动到下一个元素++$this->position=2

    string(17) "myIterator::valid"    //验证当前元素$this->array[2]是否有效, 有效则继续执行;

    string(19) "myIterator::current"  //重复之前的步骤

    string(15) "myIterator::key"      //重复之前的步骤

    int(2)                             //重复之前的步骤

    string(11) "lastelement"           //重复之前的步骤   

    string(16) "myIterator::next"      //向前移动到下一个元素++$this->position=3

    string(17) "myIterator::valid"      //验证当前元素$this->array[3]是否有效, 无效则退出遍历

通过上面的分析, **我们可以得到迭代器的工作原理**:

1. 在第一次迭代前, 先运行iterator::rewind()方法, 并且只运行一次, 返回迭代器第一个元素, 但这个方法是没有返回值的; 

2. 并且紧接着会验证第一个元素是否有效, 验证方法可以自定义, 如果返回值为true, 则继续执行, 如果返回值为false则迭代终止.

3. foreach继续调用Iterator::current()和Iterator::key(), 返回指定元素的键和值.

4. foreach的方法体. 上例中的var_dump(\$key, $value);

5. 在执行每次迭代后, 调用Iterator::next()决定下一次的元素并重复第2步.

通过迭代器iterator的5个方法, 我们实现了对遍历的灵活控制!

此外, iterator迭代器还可以作为容器, 去遍历其他类的对象, 这里就用到了聚合式迭代器IteratorAggregate{}接口

## 聚合式迭代器IteratorAggregate{}接口

所谓聚合式, 就是去实现其他迭代器功能的接口. 相当于在其他迭代器上套了个壳. 它只有一个方法:

`abstract public IteratorAggregate::getIterator()   获取一个外部迭代器`

 比如我们可以尝试将iterator{}和IteratorAggregate{}进行结合:

    class MainIterator implements Iterator
    {
        private $var = array();
        public function __construct($array)    //构造函数, 初始化对象数组
        {
            if (is_array($array)) { 
            $this->var = $array;
            }
        }

        public function rewind() {   
            echo "rewinding\n";
            reset($this->var);    //将数组的内部指针指向第一个单元
        }

        public function current() {
            $var = current($this->var);    // 返回数组中的当前值
            echo "current: $var\n";
            return $var;
        }

        public function key() {
            $var = key($this->var);       //返回数组中内部指针指向的当前单元的键名
            echo "key: $var\n";
            return $var;
        }

        public function next() {
            $var = next($this->var);     //返回数组内部指针指向的下一个单元的值
            echo "next: $var\n";
            return $var;
        }

        public function valid() {
        return !is_null(key($this->var); //判断当前单元的键是否为空
        }
    }

我们通过上面的类定义, 实现了一个迭代器MainIterator{}; 现在这个构造器就可以被IteratorAggregate{}使用:

    class MyCollection implements IteratorAggregate
    {
        private $items = array();
        private $count = 0;
        
        public function getIterator() {       //获取一个外部迭代器
            var_dump(__METHOD__);
            return new MainIterator($this->items);  //使用MainIterator接口创建实例
        }

        public function add($value) {
            $this->items[$this->count++] = $value;
        }
    }
    $coll = new MyCollection();
    $coll->add('value 1');
    $coll->add('value 2');
    $coll->add('value 3');

    foreach ($coll as $key => $val) {
        echo "key/value: [$key -> $val]\n\n";
    }
    //output:
    Collection::getIterator
    rewinding
    current: value 1
    key: 0
    key/value: [0 -> value 1]

    next: value 2
    current: value 2
    key: 1
    key/value: [1 -> value 2]

    next: value 3
    current: value 3
    key: 2
    key/value: [2 -> value 3]

    next: 
    
上面这个例子, 最关键的是这句话:

     public function getIterator() {       
            return new MainIterator(\$this->items);}
            
 翻译一下: 实例化器MainInterator构造, 并将原对象\$coll的属性\$items传给$var.
 
可以看到foreach (\$coll as \$key => $val)会自动调用getIterator()方法, 也就是调用了MainIterator构造器, 最终实现了遍历.
 
当然, 聚合型迭代器可以与很多迭代器融合, 实现更高效的迭代, 在PHP SPL标准库中有很多已经预定义好的迭代器大概20多个. 比如ArrayIterator{}, 这个迭代器允许在遍历数组和对象时删除和更新值与键。

    class myData implements IteratorAggregate {
        private $property1 = ["Public property one",'test'];

        public function getIterator() {  //创建ArrayIterator迭代器的实例, 并传属性
        return new ArrayIterator($this->property1);
        }
    }

    $obj = new myData;
    foreach($obj as $key => $value) {
        var_dump($key, $value);
    }

## 两者区别

通过之前的介绍, 我们已经知道两者在功能上有一定区别, 除此之外, 在工作模式上也有一些区别:

我们在上一节代码的基础上, 实现一个简单的递归迭代, 我们先看看mainIterator{}的处理方式:

    $list = new mainIterator([1, 2]);    //新建mainIterator对象
    foreach($list as $i){
        foreach($list as $i)
            echo $i;
            echo '\n';
    }
    //output:
    rewinding
    current: 1
    rewinding
    current: 1
    1
    next: 2
    current: 2
    2
    next: 
    next: 
 这个结果是有问题的, 在执行了第二层递归时, 数组内部指针已经到达末尾, valid()已经false, 整体强制退出了foreach, 并没有执行第二轮迭代;

 如果我们使用IteratorAggregate来调用iterator迭代器呢?
     
    $list1= new Collection();
    $list1->add(1);
    $list1->add(2);
    foreach ($list1 as $i){
        foreach ($list1 as $i)
            echo $i;
        };
    //output:
    string(23) "Collection::getIterator"
    rewinding    current: 1
    string(23) "Collection::getIterator"
    rewinding   current: 1 1
    next: 2    current: 2 2 
    next:    
    next: 2    current: 2
    string(23) "Collection::getIterator"
    rewinding     current: 1 1
    next: 2    current: 2  2
    next: 
    next: 
可以看到, **foreach通过IteratorAggregate接口成功完成了两次迭代. 差别在哪里呢?**

IteratorAggre方法在每次遇到foreach的时候, 都会重新调用getIterator方法, 也就是再整体重新跑一边迭代器iterator. 这样做的结果就是实现了嵌套, 有了递归的作用在里面. 可以满足递归迭代的需求.

而一元的iterator迭代器, 却没有这种嵌套结构, 光标到尾部, 只能退出.


**总结一下: 

通过iterator{}我们可以通过内部的5个方法, 设置各种各样的迭代器.

iterator{}可以单独用来迭代对象和数组, 但IteratorAggregate{}需要调用不同的外部迭代器.

从代码复杂度上, iterator要高于iteratorAggregator. 

执行效率上也是iteratorAggregator更高一些.

iterator{}处理递归迭代时会有问题, 而iteratorAggregate会在每次foreach运行时被调用, 更安全.

另外再补充一点, iterator构造器不仅可以被foreach调用, 还可以被while, for等循环语句都可以调用构造器的方法, 实现遍历, 相当于是手动做了个getiterator.


--------------------------------------------------------------------------


感谢阅读, 如有不准确和错误的地方请指正, 我会及时修正, 拜谢!

总结不易, 请勿私自转载, 否则别怪老大爷不客气

欢迎热爱技术的小伙伴儿和我交流, 共同成长!

参考资料:

www.php.net  PHP官方文档

[PHP interfaces IteratorAggregate vs Iterator?](https://stackoverflow.com/questions/13624639/php-interfaces-iteratoraggregate-vs-iterator)

[注意Iterator和IteratorAggregate之间的区别](https://www.codenong.com/3df7cb55a3c06bb84a70/)

