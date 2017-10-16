---
layout: post
title:  "PHP语法糖(5.6-7.1)。"
categories: laravle
tags: laravel Tp
author: 大东
---

* content
{:toc}


###类型

###Boolean

`string` 的内部结构类似 `array` , 可以像`python` 一下使用下标访问字符串
``` 
    $str = '012345';
    echo $str[1];  //1
    echo $str{2};  //2
    
```


5.4 以后可以像JS一样定义数组

```
    $arr = ['one', 'two', 'three']; //感觉方便了很多
```
###全局变量






时间长不用总会忘记重新整理一下加深下印象
####$_SERVER
```
    SERVER_ADDR IP地址 127.0.0.1
    SERVER_NAME 主机名 localhost
    SERVER_SOFTWARE 服务器类型 nginx
    REMOTE_ADDR 客户端IP. 127.0.0.1 

```
####$_FILES
```
$_FILES['file']['name'] 图片原名称
$_FILES['file']['type'] 图片MIME类型
$_FILES['file']['size'] 图片大小
$_FILES['file']['tmp_name'] 服务器端临时名称
```
####常量

- 5.3之后可以使用const来定义常量
```
    const DEBUG = true;
```
####运算符

- <=> 比较运算符，7.0 之后支持 
```
    echo  $a <=> $b;
    /*
    当 $a < $b 时， 表达式返回 -1
    当 $a = $b 时， 表达是返回 0
    当 $a > $b 时， 表达式返回  1
    */
```
- ?? 空合并运算符 PHP7特性。
```
    $name = $_POST['name'] ?? '';  //如果前面的值不null，则返回本身，否则返回后面的值

```
- 三元运算符 ?: 5.3以后可以使用
```
    $name = $_POST['name'] ?: ''; ////如果前面的值不为null，则返回本身，否则返回后面的值
    
```
####流程控制

- goto 5.3以上有效
>操作符可以用来跳转到程序中的另一位置。该目标位置可以用目标名称加上冒号来标记，而跳转指令是 goto 之后接上目标位置的标记。PHP 中的 goto 有一定限制，目标位置只能位于同一个文件和作用域，也就是说无法跳出一个函数或类方法，也无法跳入到另一个函数。也无法跳入到任何循环或者 switch 结构中。可以跳出循环或者 switch，通常的用法是用 goto 代替多层的 break。
```
    goto a;
    echo 'Foo';
    
    a:
    echo 'Bar';
    
    //输出 Bar
```

####函数
- 变长参数 ...， 5.6以后可用
```
    function dosum(...$arr){
        $sum = 0;
        foreach($arr as $v){
            $sum += $v;
        }
        return $sum;
    }
    
    $arr = [1, 2, 3, 4, 5];
    echo dosum(...$arr);   // 输出15
    echo dosum(1,2,3,4,5,6); //输出21
    
    //TODO
    /**
    这个语法比较简单。不过要注意php版本。。
    */
```
-  匿名函数（Anonymous functions）5.3
也叫闭包函数，在JS中很常见。为了防止污染全局作用域。5.3 以后PHP也支持了这种写法
```
    $test = function($name='Li'){
        echo 'My name is '.$name;
    };
    $test();
```
- 如果想要从父作用域中继承变量怎么办
```php
    //这里定义一个默认的输出名字的方式
    $tpl = 'My name is ';
    
    //使用 use() 来引用父级的变量，最后输出结果与上边一致 
    $test = function($name='Li') use($tpl) {
        echo $tpl.$name;
    };
    $test();
```
需要注意的是，闭包函数的父作用域，是定义它的作用域，不是调用的作用域

###类和对象

- ::class 类的静态方法，用于获取类的完全限定名称，（包含命名空间）
```
    namespace Foo{
        class test{
    
        }
        echo test::class; // 输出 FOO\test, 在使用命名空间的情况非常有用
    }
```
- 5.4 新增加的一个多继承实现方式trait。下面总结了一下基本概念
 1. trait 和 class 是相似的概念，但不能被实例化 
 2. 一个类可以使用多个trait，优先级是 class > trait > 父类继承的方法 
 3. 使用insteadof 来解决 tarit 冲突 
 4. 使用as，来修改方法的访问控制 
 5. 在trait中也可以使用tarit。和在class中用法一致
 
```
    trait A {
        public function say(){
            echo 'trait  A';
        }
    }
    
    trait B {
        public function say(){
            echo 'trait B';
        }
    
        public function walk(){
            echo 'walk B';
        }
    }
    
    class Person {
        use A, B{
            B :: say insteadof A; // 使用B的say方法代替了A的say方法
            walk as protected;    // 将walk 设置为受保护的
        }
    }
    
    $obj = new Person;
    $obj->say();  // echo trait A;
    $obj->walk(); // 提示不能访问一个受保护的方法
```
 6.在trait中使用， 属性、静态属性、静态方法、抽象类都是被允许的。
```
    trait Test {
        public static $obj;
        public $name = 1;
        static function createObj(){
            return empty(self::$obj) ? new self : self::$obj;
        }
    }
    
    class son {
        use Test;
    }
    
    $obj = son::createObj();
    echo $obj->name;  // echo 1
    echo $obj === $obj1 ? 0 : 1; // echo 1
```

- 5.3 类的后期静态绑定 
官方的解释是： 
>该功能从语言内部角度考虑被命名为”后期静态绑定”。”后期绑定”的意思是说，static:: 不再被解析为定义当前方法所在的类，而是在实际运行时计算的。也可以称之为”静态绑定”，因为它可以用于（但不限于）静态方法的调用 
乍一看，好像什么也没看懂。看看具体的代码吧。
```
    class A {
        public static function who() {
            echo __CLASS__;
        }
        public static function test() {
            self::who();
        }
    }
    
    class B extends A {
        public static function who() {
            echo __CLASS__;
        }
    }
    B::test(); // echo A;
    
    // 上面是一个正常的调用， 输出了 A
    
    // 当我们把 class A 的 test 方法修改一下。 将 self 改成 static 后
    
    class A {
        public static function who() {
            echo __CLASS__;
        }
        public static function test() {
            static::who();
        }
    }
    
    class B extends A {
        public static function who() {
            echo __CLASS__;
        }
    }
    B::test(); // echo B;
```
总结：`PHP5.3` 新增加了一类关键字，`static` 可以在调用函数的方法。用这个关键字，来实现了后期静态绑定。

####异常处理

比较简单记录一下
```
    try{
        throw new Execption('抛出异常');
    } catch (Execption $e){
        //获取异常
        $error = $e->getMessage();
    }
    echo $error;  //抛出异常
```