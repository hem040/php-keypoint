# php核心技能

### 基础知识

#### 1. 可变变量

在php中，可以将一个变量的值作为另一个变量的，即

~~~
$a = 'foo';
$$a = 'foo2';  //相当于 $foo = 'foo2';
echo $$a;	   // foo2
$a = 'foo3';   // 
echo $foo;     // foo2
echo $$a;      // Undefined variable: foo3
~~~

~~~
<?php
class foo {
    var $bar = 'I am bar.';
    var $arr = array('I am A.', 'I am B.', 'I am C.');
    var $r   = 'I am r.';
}

$foo = new foo();
$bar = 'bar';
$baz = array('foo', 'bar', 'baz', 'quux');
echo $foo->$bar . "\n";			\\ I am bar
echo $foo->$baz[1] . "\n";		\\ I am bar  先读取$baz[1],再读$foo->bar

$start = 'b';
$end   = 'ar';
echo $foo->{$start . $end} . "\n";  \\I am bar 相当于$foo->bar  

$arr = 'arr';
echo $foo->$arr[1] . "\n";	\\I am r.  先读取字符串$arr[1],再读取 $foo->r
echo $foo->{$arr}[1] . "\n"; \\ I am B  先读取 $foo->arr,再读取属性数组
~~~

可变变量可用于动态设置变量名。

注意：

	1. 要将可变变量用于数组，必须解决一个模棱两可的问题。这就是当写下 $$a[1] 时，解析器需要知道是想要 $a[1] 作为一个变量呢，还是想要$$a 作为一个变量并取出该变量中索引为 [1] 的值。解决此问题的语法是，对第一种情况用 ${$a[1]}，对第二种情况用 ${$a}[1]。默认情况下为${$a[1]}。
 	2. 类的属性也可以通过可变属性名来访问。可变属性名将在该调用所处的范围内被解析。例如，对于 $foo->$bar 表达式，则会在本地范围来解析 $bar 并且其值将被用于 $foo 的属性名。对于 $bar 是数组单元时也是一样。
 	3. 在 PHP 的函数和类的方法中，[超全局变量](https://www.php.net/manual/zh/language.variables.superglobals.php)不能用作可变变量。*$this* 变量也是一个特殊变量，不能被动态引用。

#### 2. 函数传值

1. 一般情况下，值传递不改变函数外部变量值

2. 如果是引用传值，则会改变函数外部的变量值

3. 函数传值严格类型：默认情况下，如果能做到的话，PHP将会强迫错误类型的值转为函数期望的标量类型。 例如，一个函数的一个参数期望是[string](https://www.php.net/manual/zh/language.types.string.php)，但传入的是[integer](https://www.php.net/manual/zh/language.types.integer.php)，最终函数得到的将会是一个[string](https://www.php.net/manual/zh/language.types.string.php)类型的值。可以基于每一个文件开启严格模式。在严格模式中，只有一个与类型声明完全相符的变量才会被接受，否则将会抛出一个**TypeError**。 唯一的一个例外是可以将[integer](https://www.php.net/manual/zh/language.types.integer.php)传给一个期望[float](https://www.php.net/manual/zh/language.types.float.php)的函数。

   ~~~
   <?php
   declare(strict_types=1);			//声明为开启严格模式
   
   function sum(int $a, int $b) {
       return $a + $b;
   }
   
   var_dump(sum(1, 2));            //int(3)
   var_dump(sum(1.5, 2.5));        //typeerror
   ~~~

   ~~~
   <?php
   function sum(int $a, int $b) {
       return $a + $b;
   }
   
   var_dump(sum(1, 2));			//int(3)
   
   // These will be coerced to integers: note the output below!
   var_dump(sum(1.5, 2.5));        int(3)
   ~~~

   4. 可变数量的参数列表

      php支持用户在自定义函数的参数中，参数列表是可变的。在php5.6+由***...***实现，在php5.5-版本中，使用函数***func_num_args()***,***func_get_arg()***,***func_get_args()***

   ~~~
   <?php
   function sum(...$numbers) {
       $acc = 0;
       foreach ($numbers as $n) {
           $acc += $n;
       }
       return $acc;
   }
   
   echo sum(1, 2, 3, 4);    //10
   ~~~

   ~~~
   <?php
   function add($a, $b) {
       return $a + $b;
   }
   
   echo add(...[1, 2])."\n";    \\ 10
   
   $a = [1, 2];
   echo add(...$a);              \\ 10
   //可变长传参给定长，取相应个数的参数
   ~~~

   ~~~
   <?php
   function foo() 
   {
   	echo 'variable number is ' . func_num_args() . "\n";
   	echo func_get_arg(1) . "\n";
   	var_dump(func_get_args());
   }
   
   foo(1,2,3);
   
   输出：
   variable number is 3
   2
   array(3) {
    [0]=>
    int(1)
    [1]=>
    int(2)
    [2]=>
    int(3)
   }
   ~~~

   5. trigger_error函数
   
   trigger_error() 函数创建用户级别的错误消息。
   
   trigger_error() 函数能结合内置的错误处理器所关联，或者可以使用用户定义的函数作为新的错误处理程序(set_error_handler())
   
   trigger_error(errormsg,errortype)
   
   ~~~
   errormsg:错误消息，required
   errortype:optional  错误类型
   					E_USER_ERROR
   					E_USER_WARNING
   					E_USER_NOTICE（默认）
   ~~~
   
   ~~~
   function foo($a) {
   	if ($a <= 10) echo $a;
   	if ($a > 10) {
   		trigger_error('var can not be gt 10'); 
   	}
   }
   
   foo(1);
   foo(12);
   
   输出：
   1
   PHP Notice:  var can not be gt 10 in /usercode/file.php on line 5
   ~~~
   
   

#### 3. 匿名函数

匿名函数（闭包），允许临时创建一个没有指定名称的函数。经常用于回调函数的参数值。

匿名函数的实现是通过php内置类  Closure来实现的

eg:

~~~
<?php
echo preg_replace_callback('~-([a-z])~', function ($match) {
    return strtoupper($match[1]);
}, 'hello-world');
// 输出 helloWorld  匹配到-w, $match值为-w，匿名函数返回W,替换后的为helloWorld
~~~

闭包函数也可以作为变量的值来使用，此时相当于实例化 Closure类给$foo对象

~~~
$foo = function($a) 
{
	echo $a;
}
$foo('I am a closures');  //I am a closures
~~~

闭包可以从父作用域中继承变量。 任何此类变量都应该用 *use* 语言结构传递进去。 PHP 7.1 起，不能传入此类变量： （预定义变量）[superglobals](https://www.php.net/manual/zh/language.variables.predefined.php)、 $this 或者和参数重名。

全局变量，存在于一个全局范围，无需继承，无论当前执行的是哪个函数。

<u>闭包从父作用于继承变量，只继承闭包定义之前的，如果变量在闭包定义后改变的，将无法继承到闭包中，当然引用除外，此时，强行改变了变量指向地址块。</u>

~~~
$message = 'hello';

// 没有 "use"
$example = function () {
    var_dump($message);
};
echo $example();		//udifined variable  未继承父类的变量

$example = function () use ($message) {
    var_dump($message);
};
echo $example();		//hello

$message = 'world';
echo $example();		//hello 想想为什么？

$example = function () use (&$message) {
    var_dump($message);
};
echo $example();		//world;

$message = 'hello';
echo $example();	   //hello    想想为什么
~~~



#### 4. 匿名类

匿名类可以创建一次性的简单对象

~~~
<?php
/*********************匿名函数************************/

$fu = function(){
    echo "这是匿名函数";
};
$fu();
echo "<br/>";

class Animal{
    
    public $num;
    public function __construct($key){
        $this->num = $key;
    }
    
    public function getValue($sum):int{
        return $this->num+$sum;
    }
}
$animal = new Animal(5);
echo $animal->getValue(10);

echo "<br/>";

/****************************匿名类***********************/
echo "这是匿名类<br/>";
echo (new class(5) extends Animal{})->getValue(90); //95
echo "<br/>";
echo (new class(5) extends Animal{})->getValue(100);  //105
~~~

注：匿名类被嵌套进普通类后，不能访问这个 外部类的 私有(private)、受保护(protected)方法或属性。但如果想访问protected方法或属性，可以 继承（extends）这个外部类，想访问这个 私有（private）方法或属性，可以通过构造器，如下代码所示:

~~~
<?php
class Animal{
    private $num = 1;
    
    protected $age = 2;
    
    protected function bark(){
        return 10;
    }
    
    public function drive(){
        return new class($this->num) extends Animal{  //匿名类访问普通类的私有属性通过构造器
            protected $id;
            
            public function __construct($sum){   
                $this->id = $sum;
            }
            
            public function eat(){
                return $this->id+$this->age+$this->bark();  //访问私有类的受保护方法和属性，通过继承
            }
        };
    }
}

echo (new Animal)->drive()->eat();  //13
~~~



### 面向对象

#### 类self和static区别

1. php中self只会调用当前类的方法，即self不会调用子类的方法。

2. static关键字和延迟绑定：在php5.3中加入了一个延迟静态绑定的新特性，它可以帮我们实现多态。简单说，延迟静态绑定意味着我们用static关键字调用一个被继承的方法时，它将在运行时绑定调用类。

   eg:

   ~~~~~~~~~~~
   class Car
   {
       public static function model()
       {
            self::getModel();
       }
   
       protected static function getModel()
       {
           echo "I am a Car!";
       }
   
   }
   ~~~~~~~~~~~

   此时我们调用静态方法

   ~~~~~~
   Car::model()   //输出 I am a Car
   ~~~~~~

   我们添加一个新类，继承Car这个类

   ~~~~
   class BMW extends Car
   {
   	public static function getModel()
   	{
   		echo "I am a BMW!"
   	}
   }
   ~~~~

   此时我们再调用静态方法

   ~~~
   BMW::model()   //按照正常继承逻辑，应该输出  I am a BMW!，最后输出结果却是 I am a Car!
   ~~~

   这就是因为php中self只会调用当前类的方法，若改为：

   ~~~
   class Car
   {
       public static function model()
       {
            static::getModel();
       }
   
       protected static function getModel()
       {
           echo "I am a Car!";
       }
   
   }
   ~~~

   ~~~
   此时static延迟了静态绑定，BMW类调用model方法时，运行在绑定类BMW,输出  I am a BMW!
   ~~~

   ~~~
   class A 
   {
       public function create1()
       {
           return new self();
       }
   
       public function create2()
       {
           return new static();
       }
   }
   
   class B extends A {
   
   }
   
   $objA = new A();
   echo get_class($objA->create1());
   echo get_class($objA->create2());
   echo "<br/>";
   $obj = new B();
   echo get_class($obj->create1());
   echo get_class($obj->create2());
   
   显示结果如下:
   AA
   AB
   ~~~

   ~~~
   总结：
   self - 就是这个类，是代码段里面的这个类。 
   static - PHP 5.3加进来的只得是当前这个类，有点像$this的意思，从堆内存中提取出来，访问的是当前实例化的那个类，那么 static 代表的就是那个类。
   ~~~

   #### 面向对象关键字

   #### traits代码复用
   
   php一直都是单继承的，无法同时从两个基类中继承属性和方法，为了解决这个问题，php从5.4开始，实现了一种代码复用的方法，即trait。
   
   trait和class类似，但是trait仅仅是用细粒度和一致的方式来组合功能。无法通过trait来实现实例化。trait为传统的继承之外增加了水平特性的组合。
   
   eg：
   
   ~~~
   <?php
   trait ezcReflectionReturnInfo {
       function getReturnType() { /*1*/ }
       function getReturnDescription() { /*2*/ }
   }
   
   class ezcReflectionMethod extends ReflectionMethod {
       use ezcReflectionReturnInfo;
       /* ... */
   }
   
   class ezcReflectionFunction extends ReflectionFunction {
       use ezcReflectionReturnInfo;
       /* ... */
   }
   ~~~
   
   ###### 优先级
   
   如果当前类B继承了父类A的method方法，同时引用了 trait C(包含method方法)，B又定义了method方法，当  b = new B()时，优先级顺序
   
    ~~~
   B.method > C.method > A.method
    ~~~
   
   即 <u>从基类继承的成员会被 trait 插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了 trait 的方法，而 trait 则覆盖了被继承的方法</u>。
   
   ###### 多个trait
   
   一个类中要引用多个trait A、trait B，在use声明中用逗号隔开，即
   
   ~~~
   use A,B;
   ~~~
   
   思考：以下程序会输出什么结构
   
   ~~~
   <?php
   	trait A 
   	{
   		public function method() 
   		{
   			echo 'A';
   		}
   	}
   	trait B
   	{
   		public function method()
   		{
   			echo 'B';
   		}
   	}
   	class Mytest
   	{
   		use A,B;
   	}
   	$obj = new Mytest();
   	$obj->method();
   ~~~
   
   最后输出结果
   
   ![](.\images\trait_1.png)
   
   即引用的两个trait下如果有同名方法，则会产生以上致命错误，如果没有引用到一个类中，则不会产生以上冲突和错误
   
   ###### 解决以上冲突的方法
   
   为了解决多个 trait 在同一个类中的命名冲突，需要使用 *insteadof* 操作符来明确指定使用冲突方法中的哪一个。以上方式仅允许排除掉其它方法，*as*操作符则可以 为某个方法引入别名。 注意，*as* 操作符不会对方法进行重命名，也不会影响其方法。
   
   ~~~
   <?php
   trait A {
       public function smallTalk() {
           echo 'a';
       }
       public function bigTalk() {
           echo 'A';
       }
   }
   
   trait B {
       public function smallTalk() {
           echo 'b';
       }
       public function bigTalk() {
           echo 'B';
       }
   }
   
   class Talker {
       use A, B {
           B::smallTalk insteadof A;
           A::bigTalk insteadof B;
       }
   }
   
   class Aliased_Talker {
       use A, B {
           B::smallTalk insteadof A;
           A::bigTalk insteadof B;
           B::bigTalk as talk;
       }
   }
   
   $talker = new Talker();
   $talker->smallTalk();
   $talker->bigTalk();
   $as_talker = new Aliased_Talker();
   $as_talk->bigTalk();
   $as_talk->smallTalk();
   $as_talk->talk();
   ~~~
   
   以上结果输出：
   
   ![](.\images\trait_2.png)

###### 在引用类中修改引用trait中方法的控制

可在引用类中，对引用的trait方法，修改方法的访问控制，如下：

~~~
<?php
trait A {
    public function methodA() {
        echo 'a';
    }
	
	 public function methodB() {
        echo 'B';
    }
}

class MyTestA
{
	use A{methodA as public;}
}

class MyTestB
{
	use A{methodA as private myPrivateMethodA;}
}

class MytestC extends MyTestA
{
}

class MytestD extends MyTestB
{
}

$objA = new MytestA();
$objA->methodA();								//输出 a
~~~

###### 从 trait 来组成 trait

#### 设计模式

##### 创建型设计模式

​	概念：专注于如何创建对象以及对象组 

###### 1. 单例模式

​	概念：单例模式是一种软件设计模式，它将类的实例化限制为一个对象。当需要一个对象来协调整个系统的操作时，这非常有用。

​	要创建单例，需禁用构造方法，禁用克隆，禁用扩展，并创建静态变量以容纳实例。

~~~
class RpcClient() 
{
	private static $instance;
	public function __construct() 
	{
		
	}
	
	public function __clone() 
	{
	
	}
	
	public function __wakeup() 
	{
	
	}
	
	public static function getInstance() 
	{
		if (!self::$instance) {
			self::$instance = new self();
		}
		return self::$instance;
	}
}
~~~



​	单例模式

1. 懒汉

   懒汉模式即在第一次调用时实例化自己。即上面这种模式。

  2. 饿汉

     在类被加载时，已提前创建好实例并在静态变量$instance中：

     ~~~
     2 class RpcClient() 
     3 {
     4	private static $instance = new self();  //这么写会报语法错误，原理是这样的
     	public function __construct() 
     	{
     		
     	}
     	
     	public function __clone() 
     	{
     	
     	}
     	
     	public function __wakeup() 
     	{
     	
     	}
     	
     	public static function getInstance() 
     	{
     		if (!self::$instance) {
     			self::$instance = new self();
     		}
     		return self::$instance;
     	}
     }
     
     PHP Parse error:  syntax error, unexpected T_NEW in /usercode/file.php on line 4
     ~~~

     附：一个redis的单例模式

     ~~~
     <?php
     class redis
     {
         const REDISHOST = '127.0.0.1';
         const REDISPORT = '6379';
         const REDISPASWORD = '';
         const REDISDBNAME = 0;
         private static $_obj = null;
         private function __construct(){
         }
         private function __clone(){}//禁止克隆
         private  static function connect_redis($dbname = null)
         {
             try{
                 self::$_obj = new redis();
                 self::$_obj->connect(self::REDISHOST,self::REDISPORT);
                 if(self::REDISPASWORD){
                     self::$_obj->auth(self::REDISPASWORD);
                 }
                 if($dbname){
                     $dbname = (int)$dbname;
                     self::$_obj->select($dbname);
                 }else{
                     self::$_obj->select(self::REDISDBNAME);
                 }
             }catch (Exception $e){
                 exit($e->getMessage().'<br/>');
             }
             return self::$_obj;
         }
         public static function getRedis()
         {
             if(!self::$_obj){
                 self::$_obj = self::connect_redis();
             }
             return self::$_obj;
         }
         public function set($key,$value)
         {
             if(!empty($key) && !empty($value)){
                 return self::$_obj->set($key,$value);
             }else{
                 return false;
             }
         }
         public function get($key)
         {
             if(!empty($key)){
                 return self::$_obj->get($key);
             }else{
                 return false;
             }
         }
         public function exists($key)
         {
             if(!empty($key)){
                 return self::$_obj->exists($key);
             }else{
                 return false;
             }
         }
     }
     //直接调用getredis
     $redis =  redis::getredis();
     $redis->get('a');
     ~~~

     

###### 2. 简单工厂模式

​	概念：只是为客户端生产实例，而不向客户端公开实例化逻辑

~~~
interface Door
{
   public function getWidth():float;
   public function getHeight():float;
}

class houseDoor implements Door 
{
    protected $Height = 0;
    protected $Width = 0;
    
    public function __construct($width = 0, $height = 0) 
    {
        $this->Height = $height;
        $this->Width = $width;
    }
    
    public function getWidth():float
    {
        return $this->Width;
    }
    public function getHeight():float
    {
        return $this->Height;
    }
}

class DoorFactory
{
    public function makeDoor($width = 0, $height = 0)
    {
        return new houseDoor($width, $height);
    }
}

$objB = new DoorFactory();
$objA = $objB->makeDoor(200, 400);
echo $objA->getWidth();
echo "\n";
echo $objA->getHeight();
~~~



###### 3.工厂方法模式

###### 4. 抽象工厂模式

###### 5. 原型模式

###### 6. 构建器模式

##### 结构型设计模式

###### 1. 适配器模式

###### 2. 桥梁模式

###### 3. 组合模式

###### 1. 适配器模式

###### 4. 装饰模式

###### 5. 门面模式

###### 6. 享元模式

###### 7. 代理模式

##### 行为型设计模式

###### 1. 责任链模式

###### 2. 命令行模式

###### 3. 迭代器模式

###### 4. 中介者模式

###### 5. 备忘录模式

###### 6. 观察者模式

###### 7. 访问者模式

###### 8. 策略模式

###### 9. 状态模式

###### 10. 模板方法模式

### 核心知识

#### 调试工具

##### xdebug的安装与使用

很多PHP程序员调试使用echo、print_r()、var_dump()、printf()，通过分段输出，来定位程序问题，这样调试，效率比较低下，想要实时了解程序的输出且连续的分步调试，这时候我们就需要用到xdebug。Xdebug是一个开放源代码的PHP程序调试器(即一个Debug工具)，可以用来跟踪，调试和分析PHP程序的运行状况。

###### 安装步骤

~~~
1.下载：官网：https://xdebug.org/download.php  
  注：linux下载source文件
  	  windows下载.dll文件
  	  需要和相关的php版本匹配
2. 安装：
   wget https://xdebug.org/files/php_xdebug-2.8.0alpha1-7.1-vc14-nts-x86_64.dll
   tar -xvzf xdebug-2.8.0alpha1.tgz
   cd xdebug-2.8.0alpha1
   phpize
   ./configure --with-php-config=/usr/local/php71/bin/php-config (where is php-config可查找到相关路径)
   make && make install
~~~

安装成功，则会出现如下信息，进出xdebug.so的目录位置

![](G:\study\技能考核\php-\images\174510_nRXs_1761919.jpg)

###### 配置

~~~
修改php.ini，加入如下信息
[xdebug]
zend_extension="/usr/local/php71/lib/php/extensions/no-debug-non-zts-20160303/xdebug.so"
xdebug.remote_enable = 1
xdebug.remote_handler = "dbgp"
xdebug.remote_host = "192.168.29.168"
xdebug.remote_port = 9009
xdebug.idekey = "PHPSTORM"
xdebug.trace_output_dir = "/tmp/phptrace"
xdebug.collect_vars = 1
xdebug.collect_return = 1
xdebug.collect_params = 1
xdebug.remote_autostart = 1
xdebug.show_exception_trace = 1
~~~

配置完成后，重启php-fpm，开启9009端口，此时虚拟机配置完成，查看phpinfo，显示如下，代表安装成功

![](.\images\1563352213(1).png)

###### phpstorm配置

对phpstorm分别做如下配置

![](.\images\debug_set.jpg)

![](.\images\server_set.jpg)

###### chrome浏览器

对chrome浏览器安装Xdebug helper，并做如下设置

![](.\images\xdebug_helper.jpg)

此时用chrome请求服务时，点击phpstorm监听

![](.\images\lisen_set.jpg)

###### postman配置

在请求头

headers加上key值为Cookie，value为XDEBUG_SESSION=PHPSTORM，phpstorm同样打开监听即可。