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
   
   

### 核心知识

### 调试工具

####		xdebug的安装与使用

很多PHP程序员调试使用echo、print_r()、var_dump()、printf()，通过分段输出，来定位程序问题，这样调试，效率比较低下，想要实时了解程序的输出且连续的分步调试，这时候我们就需要用到xdebug。Xdebug是一个开放源代码的PHP程序调试器(即一个Debug工具)，可以用来跟踪，调试和分析PHP程序的运行状况。

##### 安装步骤

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

##### 配置

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

![](G:\study\技能考核\php-\images\1563352213(1).png)

##### phpstorm配置

对phpstorm分别做如下配置

![](G:\study\技能考核\php-\images\debug_set.jpg)

![](G:\study\技能考核\php-\images\server_set.jpg)



##### chrome浏览器

对chrome浏览器安装Xdebug helper，并做如下设置

![](G:\study\技能考核\php-\images\xdebug_helper.jpg)

此时用chrome请求服务时，点击phpstorm监听

![](G:\study\技能考核\php-\images\lisen_set.jpg)



##### postman配置

在请求头

headers加上key值为Cookie，value为XDEBUG_SESSION=PHPSTORM，phpstorm同样打开监听即可。