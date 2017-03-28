---
layout: post
title:  "func_get_args和func_num_args详解"
categories: PHP
tags: php函数
author: 大东
---

* content
{:toc}

`func_get_args()`—返回的是一个数组，这个数组内的每一项都是函数的一个参数。根据php手册我们给出函数的用法格式。

```
array func_get_args ( void )
```
如果只是在这里笼统的阐述，可能大家不能够真正的了解这个函数，那么就让我们通过事例来看下这个函数的用法。
```
function foo() {     
 $args = func_get_args();   
 foreach ($args as $k => $v) {  echo “arg”.($k+1).”: $v\n”;
  }  
}  
foo();  /* 没用任何输出*/  
foo(‘hello’);  /* 输出  arg1: hello  */ 
foo(‘hello’, ‘world’, ‘again’);  /*输出 arg1: hello  arg2: world  arg3: again  */
```
这个函数可以把你传入的所有参数全部都放在一个数组中，然后再输出。这样对我们以后编写php程序是不是又简单了许多呢？既然说到了`func_get_args`函数,那么我们就不能不提下`func_num_args`函数和`func_get_arg`函数了





`func_nums_args`——统计传入函数参数的个数

`func_get_arg`——根据索引取得某一个参数，这里的索引数传入函数的参数

我们就以php手册上的例子来看吧
```
<?
 function foo(){    $numargs = func_num_args();    echo “Number of arguments: $numargs\n“;
}
foo(1, 2, 3);    // Prints ‘Number of arguments: 3′?>
```

 

上面的例子很明白的给我们展示了`func_num_args`函数就是活的传入函数的参数
```
<?php
    function foo()
    {         $numargs = func_num_args();         echo "Number of arguments: $numargs<br />\n";         if ($numargs >= 2) {         echo "Second argument is: " . func_get_arg(1) . "<br />\n";
         }
    }
     
    foo (1, 2, 3);    //Prints
    //Number of arguments: 3
    //Second argument is: 2
    ?>
```
    
上面的例子中`func_get_arg(1)`就是获取函数的第二个参数。好了，我们看下这三个函数的综合实例吧，这样我们就可以把这三个函数掌握了。
```
<?phpfunction foo(){    $numargs = func_num_args();//得到参数的个数
    echo "Number of arguments: $numargs<br />\n";    if ($numargs >= 2) {        echo "Second argument is: " . func_get_arg(1) . "<br />\n";
    }    $arg_list = func_get_args();    for ($i = 0; $i < $numargs; $i++) {        echo "Argument $i is: " . $arg_list[$i] . "<br />\n";
    }
}

foo(1, 2, 3);
/*Number of arguments: 3
Second argument is: 2
Argument 0 is: 1
Argument 1 is: 2
Argument 2 is: 3*/
?>
```
