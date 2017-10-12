---
layout: post
title:  "在tp框架中使用composer,并移值laravel框架的DUMP()和DD()函数。"
categories: laravle
tags: laravel Tp
author: 大东
---

* content
{:toc}



最近公司的项目使用的是TP3.1.本地环境安装起来各种不爽。之前使用 `Laravel` ，本地环境是 `Homestead` 和 `Docker` ， `PHP` 版本也是7以上。但是 `TP3.1` 本身就不支持 `PHP7` ，而且需要安装一些其它的扩展。所以我的本地环境只是全部重新安装，最后只能只用集成工具 `MAMP` , `PHP` 版本也只能使用 `PHP 5.4` 了，使用回 `TP` 后，最不习惯的首先就是没有 `dd` 了，没关系 ，我们来动手把 `dd` 移值过来就好啦，大家都知道 `Tp5` 以下的版本默认不支持 `Composer` ，没关系，我们来让他支持不就好啦！
#### 首先安装 `Composer`

在根目录下新建 `composer.json` 文件：
```
{
  "name": "dong/tp3.1_with_composer",
  "description": "tp3.1 with composer",
  "type": "framework",
  "keywords": ["framework","TP_with_composer","ORM"],
  "homepage": "http://qiehe.net/",
  "license": "Apache2",
  "authors": [
    {
      "name": "jourdon",
      "email": "jourdon@qiehe.net"
    }
  ]
}

```
然后执行
``` 
    composer init
```






我们会看到根目录下出现了 `vendor`, 接下来。需要在TP中加载  `Composer` 中的`autoload.php` 文件，记得要在入口文件中 `require`.

```
//开启调试模式
    define('APP_DEBUG', true);
    //加载composer,根据你的入门文件更改地址
    require '../vendor/autoload.php';
    //加载框架入口文件
    require '../ThinkPHP/ThinkPHP.php';
```
Ok,现在随便安装 `Composer` 扩展包了。
#### 安装扩展包实现DD函数
接下来我们来加入 `Laravel` 中的 `dd` 函数， `dd` 其实是 `Laravel` 安装的一个扩展包实现的，我们只需要安装这个扩展就OK了。


``` 
    composer require symfony/var-dumper
```
安装完以后还不可以。应该 `symfony/var-dumper` 这个扩展包使用了 `dump` 函数，而 `Thinkphp` 本身也有这个函数，所以我们需要在TP中对 `dump` 函数做出判断，找到 `Think/Common/functions.php` ,再找到 `dump` 方法，做一个判断就好了。
``` 
    //如果已经存在这个函数就不重新定义dump了
    if (! function_exists('dump')) {
        function dump($var, $echo=true, $label=null, $strict=true) {
            //代码就不抄过来了
    }
```
别着急，在最下面再加上 `dd` 函数
``` 
    //定义dd方法
    if (! function_exists('dd')) {
        function dd($args)
        {
            dump($args);
            die();
        }
    }
```
其实只是在 `dump` 的基础上加了一个 `die` 而已，这时候，其实 `dd` 和 `dump` 都已经可以用了。
随便写个代码输出一下。
```
    dump(['result'=> "200",'data'=> $date]);
    dd(['result'=> "200",'data'=> $date]);
```
效果如下

![](http://www.qiehe.net/image/WX20171012-142625@2x.png)

你会发现个问题，样式和 `Laravel` 里不一样啊？没关系。改下样式就好啦，找到文件 `vendor/symfony/var-dumper/Dumper/HtmlDumper.php`，修改如下代码：

``` 
        protected $styles = array(
            'default' => 'background-color:#18171B; color:#FF8400; line-height:1.2em; font:12px Menlo, Monaco, Consolas, monospace; word-wrap: break-word; white-space: pre-wrap; position:relative; z-index:99999; word-break: normal',
            'num' => 'font-weight:bold; color:#1299DA',
            'const' => 'font-weight:bold',
            'str' => 'font-weight:bold; color:#56DB3A',
            'note' => 'color:#1299DA',
            'ref' => 'color:#A0A0A0',
            'public' => 'color:#FFFFFF',
            'protected' => 'color:#FFFFFF',
            'private' => 'color:#FFFFFF',
            'meta' => 'color:#B729D9',
            'key' => 'color:#56DB3A',
            'index' => 'color:#1299DA',
        );
        
        //修改成以下代码
        
        protected $styles = array(
                    'default' => 'background-color:#fff; color:#222; line-height:1.2em; font-weight:normal; font:12px Monaco, Consolas, monospace; word-wrap: break-word; white-space: pre-wrap; position:relative; z-index:100000',
                            'num' => 'color:#a71d5d',
                            'const' => 'color:#795da3',
                            'str' => 'color:#df5000',
                            'cchr' => 'color:#222',
                            'note' => 'color:#a71d5d',
                            'ref' => 'color:#a0a0a0',
                            'public' => 'color:#795da3',
                            'protected' => 'color:#795da3',
                            'private' => 'color:#795da3',
                            'meta' => 'color:#b729d9',
                            'key' => 'color:#df5000',
                            'index' => 'color:#a71d5d',
                );
```
果然，效果和 `Laravel` 下是一样的

![](http://www.qiehe.net/image/WX20171012-143918@2x.png)

OK。完成了。。。

等下，有人会说了，为什么可以打印两个出来？

好吧，我再说一下， `symfony/var-dumper` 中的 `dump` 其实就是没有 `die` 效果的 `dd`,注意看我的 `dd` 函数 。
``` 
    function dd($args)
            {
                dump($args);
                die();
            }
```

所以我只是在 `dump` 的基础上加了一个 `die` 而已。

所以调试时需要 `die` 掉的就用 `dd` ，不需要 `die`掉的就直接用 `dump` 就好了。再也不用担心调试 `foreach` 时麻烦了。

Happy ending！