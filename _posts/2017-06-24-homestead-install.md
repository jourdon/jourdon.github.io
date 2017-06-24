---
layout: post
title:  "laravel-homestead 开发环境安装"
categories: laravel
tags: laravel homestead
author: 大东
---

* content
{:toc}


#### 准备工作

`vagrant`,`virtualBos`是必须要有的软件，
同时，你的电脑应该已经具备如下环境：`Composer`, `git`, `php`（可选)

你的电脑最好是能够畅游互联网，可以访问 github.com。

#### 安装
```
    vagrant box add laravel/homestead
```
本土局域网用户也许会在安装中遇到下载失败的问题，可以先离线下载安装包到本地，然后使用如下命令安装 
```
    vagrant box add laravel/homestead  f://homestead.box
```

#### homestead 命令行管理虚拟机

首先安装`homestead`
```
composer global require "laravel/homestead=~2.0"

```
然后，你的`~/.composer/vendor/bin`目录下应该是具备了`homestead`文件，该文件具有执行权限。

##### 添加环境变量（可能成为你的绊脚石）

下面修改环境变量，将`~/.composer/vendor/bin`写入`PATH`环境变量里。
`cmd`中使用`echo %PATH%`可以看到现在的环境变量，使用`set PATH=`即可写入环境变量而不需要重启电脑







```
F:\homestead>homestead
Laravel Homestead 2.2.2

Usage:
  command [options] [arguments]

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands:
  destroy     Destroy the Homestead machine
  edit        Edit a Homestead file
  halt        Halt the Homestead machine
  help        Displays help for a command
  init        Create a stub Homestead.yaml file
  list        Lists commands
  make        Install Homestead into the current project
  provision   Re-provisions the Homestead machine
  resume      Resume the suspended Homestead machine
  run         Run commands through the Homestead machine via SSH
  ssh         Login to the Homestead machine via SSH
  ssh-config  Outputs OpenSSH valid configuration to connect to Homestead
  status      Get the status of the Homestead machine
  suspend     Suspend the Homestead machine
  up          Start the Homestead machine
  update      Update the Homestead machine image
```
如果`homestead`命令后是上面的样了了。那么`homestead`安装成功并且环境变量成效了。

现在我们应该可以用`homestead`命令了，可以看到`homestead`包含了一些`vagrant`的功能，下面初始化：

```
    homestead init
```
此时应会在~/.homestead/下创建配置文件`Homestead.yaml`，如果该文件已经存在会问你是否覆盖。如果想编辑配置文件就输入`homestead edit`,当然，这需要在`LINUX`下。。

配置`~/.homestead/Homestead.yaml`：


说明：
```
//目录映射
folders:
    - map: F:\homestead\html   //本地目录
      to: /home/vagrant/html   //虚拟机上的目录

//虚拟目录

sites:
    - map: homestead.app    //域名，自己的 hosts 文件里已经定义了的。
      to: /home/vagrant/html/Laravel/public  //虚拟主机里的目录，一般这个目录是通过目录映射映射好了的。
    - map: test.local
      to: /home/vagrant/Code/test
    - map: hfcms.local
      to: /home/vagrant/Code/hfcms/public
    - map: flyer.dev
      to: /home/vagrant/Code/flyer.dev/public
```
新增或修改虚拟目录配置Homestead.yaml后，就可以启动`homestead`了。
```
    homestead up
```
现在可以通过`homestead.app`来访问网站了。前提是你已经下载好了`laravel`

这里有个坑，来填一下。。大部份在这里直接访问`homestead.app`会报502错误，这是因为`homestead`内配置的`BUG`.

#####  解决过程记录

查看错误日志 `/var/log/nginx/<domain>.log`
错误信息如下：
```
connect() to unix:/var/run/php5-fpm.sock failed (2: No such file or directory) while connecting to upstream, client: 192.168.10.1, server: laravel.test.app, request: “GET / HTTP/1.1”, upstream: “fastcgi://unix:/var/run/php5-fpm.sock:”, host: “laravel.test.app”
```
分析错误信息，发现是没有php5-fpm.sock这个文件

查看php版本，当前环境中安装的是PHP7.1
查看fpm的配置文件,发现了如下错误。
```
    pid = /run/php/php7.1-fpm.pid
    //目录错误，修改为
    pid = /var/run/php/php7.1-fpm.pid

```
修改nginx配置文件，
```
 fastcgi_pass unix:/var/run/php5-fpm.sock; 
    //修改为 
fastcgi_pass unix:/run/php/php7.0-fpm.sock;
```
重启`nginx`和`PHP`，搞定了。