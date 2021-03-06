---
layout: post
title:  "使用 Supervisor 管理 Laravel 队列"
categories: Laravel
tags: Laravel Supervisor queue
author: 大东
---

* content
{:toc}


### 安装
我们这里只介绍`centos`安装方式

```
    yum install python-setuptools
    easy_install supervisor
```
如果有如下结果，说明有错误，没有安装成功
``` 
    Couldn't find index page for 'meld3' (maybe misspelled?)
    Scanning index of all packages (this may take a while)
    Reading http://pypi.python.org/simple/
    No local packages or download links found for meld3>=0.6.5
    error: Could not find suitable distribution for Requirement.parse('meld3>=0.6.5')
```
去安装 `meld3` 
``` 
    wget https://pypi.python.org/packages/0f/5e/3a57c223d8faba2c3c2d69699f7e6cfdd1e5cc31e79cdd0dd48d44580b50/meld3-1.0.1.tar.gz#md5=2f045abe0ae108e3c8343172cffbe21d
    tar -xf meld3-1.0.1.tar.gz
    cd meld3-1.0.1
    python setup.py install
```
再去执行 `easy_install supervisor` ，此时的结果显示；
``` 
    Processing dependencies for supervisor
    Finished processing dependencies for supervisor
```
安装成功，继续往下吧

### `Supervisor`的配置#

运行这个命令可以生成一个默认的配置文件：
```
    echo_supervisord_conf > /etc/supervisord.conf
```
生成成功后，打开编辑这个文件，把最后的`include`块的注释打开，并修改如下：
```
    [include]
    files = /etc/supervisor/*.conf
```
新增的`Supervisor` 配置文件放在 `/etc/supervisor` 目录下，并且以 conf 结尾。


#### 使用`Supervisor`管理`Laravel`队列进程#

我们使用`Laravel`队列，会用到`php artisan queue:work`命令，让它监听队列，我们可以通过 `nohup`方式让它在后台运行，但是进程如果意外中断是不会自动重启的，所以使用`Supervisor`来监控进程是个很好的方式。

首先在`/etc/supervisor`目录下新增一个`Supervisor`的配置文件，如下：





```
    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/wwwroot/site.webshowu.com/artisan queue:work --daemon --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=www
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/var/log/supervisor/laravel-queue.log

```
这里的`command`后的`/home/wwwroot/site.webshowu.com`是站点目录,
`user`填写网站运行进程的用户，如`vagrant`,`numprocs`表示启动多少个进程来监听`Laravel`队列。
`stdout_logfile`后面的是日志目录。记得去新建这个文件，用户组为`user`的用户组，另外把日志文件权限设置为`777`。
一切就绪后，我们使用如下命令就可以启动队列进程的监听了：

#### 启动`Supervisor`：
```
    supervisord -c /etc/supervisord.conf
    supervisorctl start laravel-worker:*
```
#### 关闭`Supervisor`

```
    supervisorctl shutdown
```
#### 重新载入配置
```
    supervisorctl reload

```
    