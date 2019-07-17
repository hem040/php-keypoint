# php核心技能

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