---
layout: post
title:  "Laravel自定义全局常量和全局公用方法"
categories: Laravel
tags: Laravel
author: 大东
---

* content
{:toc}


1、在`app\Common\`目录下，或者任意你喜欢的位置创建`funcion.php`，里面写上你的全局方法
```
//获取当前文章ID的分类名
function get_cate_name($id)
{
   $cate=\App\Http\Model\Category::find($id);
   return $cate->cate_name;
}
```
2、在`composer.json`中的`autoload`中添加`files`
```
"autoload": {
    "files":[
        "app/Common/function.php"
    ]
},
```
3、重新生成`autoload`
```
composer dumpautoload
```
4、到`vendor/composer/autoload_files.php`中，查看`function`是否加载
```
return array(
    '0e6d7bf4a5811bfa5cf40c5ccd6fae6a' => $vendorDir . '/symfony/polyfill-mbstring/bootstrap.php',
    '1d1b89d124cc9cb8219922c9d5569199' => $vendorDir . '/hamcrest/hamcrest-php/hamcrest/Hamcrest.php',
    '667aeda72477189d0494fecd327c3641' => $vendorDir . '/symfony/var-dumper/Resources/functions/dump.php',
    'bd9634f2d41831496de0d3dfe4c94881' => $vendorDir . '/symfony/polyfill-php56/bootstrap.php',
    '2c102faa651ef8ea5874edb585946bce' => $vendorDir . '/swiftmailer/swiftmailer/lib/swift_required.php',
    'e7223560d890eab89cda23685e711e2c' => $vendorDir . '/psy/psysh/src/Psy/functions.php',
    '5255c38a0faeba867671b61dfda6d864' => $vendorDir . '/paragonie/random_compat/lib/random.php',
    'f0906e6318348a765ffb6eb24e0d0938' => $vendorDir . '/laravel/framework/src/Illuminate/Foundation/helpers.php',
    '58571171fd5812e6e447dce228f52f4d' => $vendorDir . '/laravel/framework/src/Illuminate/Support/helpers.php',
    '4a1f389d6ce373bda9e57857d3b61c84' => $vendorDir . '/barryvdh/laravel-debugbar/src/helpers.php',
    '3484a5f6916149a513b73c42925ec79a' => $baseDir . '/app/Common/function.php',
);
```
最下面一行可以看到`function.php`已经成功加载，

搞定收工

