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

