---
layout: post
title:  "解决ssh链接服务器超时自动断开的问题"
categories: Linux
tags: Linux Centos
author: 大东
---

* content
{:toc}

默认情况下，为了安全性；ssh的连接超时时间很短，可能我去看个文档、发个呆的时间就断开了。
下次又要重新连接，好麻烦的说。

事实上是超时时间是可以设置的。

示例环境：服务器：centos7

首先备份配置文件,养成`备份配置项`的好习惯


```
    [root@sdfsef /]cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak 
```

 编辑配置


![](http://www.qiehe.net/image/20170330150359.jpg)



```
    [root@sdfsef /]vim /etc/ssh/sshd_config    
    删除110行ClientAliveInterval 0 前面的注释#号 并改为ClientAliveInterval 5
    删除111行的lientAliveCountMax 3 前面的注释#号 并改为ClientAliveCountMax

```
                                        
重启ssh服务；

``` 
    [root@sdfsef /]service sshd reload;
    [root@sdfsef /]/etc/init.d/sshd restart;
```

搞定收工
