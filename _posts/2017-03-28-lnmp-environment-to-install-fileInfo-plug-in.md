---
layout: post
title:  "Laravel上传图片报错？你可能没有安装fileinfo插件"
categories: Laravel
tags: Laravel lnmp
author: 大东
---

* content
{:toc}


做了一个上传图片的功能，在本地上传通过，服务器报bug

```
Unable to guess the mime type as no guessers are available(Did you enable the php_fileinfo extension?)
```
主要原因是php_fileinfo没有安装

解决#

找到lnmp下的PHP安装源位置：

```
/home/lnmp1.3/src/php-5.6.22
   
```
进入fileinfo插入目录

```
cd /home/lnmp1.3/src/php-5.6.22/ext/fileinfo/
```

运行phpize
```
/usr/local/php/bin/phpize
```


返回类似信息

```
Configuring for:
PHP Api Version:         20041225
Zend Module Api No:      20060613
Zend Extension Api No:   220060519
```
依次执行下例命令 (注意这是两条命令!)
```
./configure --with-php-config=/usr/local/php/bin/php-config
 
make && make install
```
返回类似信息

```
Build complete.
Don't forget to run 'make test'.
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/
```
表明安装成功了

这时去 `/usr/local/php/etc/php.ini`文件中加入扩展

```
extension=fileinfo.so
```
重启 php服务或者直接重启LNMP

```
lnmp restart
```
查询是否开启成功 

```
/usr/local/php/bin/php -m
```
(此命令显示目前已经安装好的PHP模块)

```
[root@iZwz9 php-5.6.22]#  /usr/local/php/bin/php -m 
[PHP Modules]
bcmath
Core
ctype
curl
date
dom
ereg
fileinfo
```
可以看到fileinfo已经出现了

收工大吉
