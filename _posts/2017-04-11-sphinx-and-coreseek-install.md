---
layout: post
title:  "sphinx及coreseek的安装与配置"
categories: sphinx
tags: sphinx 
author: 大东
---

* content
{:toc}

`sphinx`是国外的一款搜索软件。

`coreseek`是在`sphinx`的基础上，增加了中文分词功能，换句话说，就是支持了中文。

Coreseek发布了3.2.14版本和4.1版本，其中的3.2.14版本是2010年发布的，它是基于Sphinx0.9.9搜索引擎的。而4.1版本是2011年发布的，它是基于Sphinx2.0.2的。Sphinx从0.9.9到2.0.2还是有改变了很多的，有很多功能，比如sql_attr_string等是在0.9.9上面不能使用的。

可以同时安装`sphinx`，`coreseek`，不会互相冲突。

环境：`centos6.5 + mysql5.6 + other`。
 
### 首先安装sphinx：

1. 下载安装包 http://sphinxsearch.com/downloads/    (目前最新版本是2.2.10)

2.  ./configure --prefix=/usr/local/sphinx --with-mysql=/usr/local/mysql        (mysql其实不用指定，默认已经支持)          
    make && make install

3. 安装完成后，在 /usr/local/sphinx目录下会有4个目录 

![](http://images2015.cnblogs.com/blog/807718/201607/807718-20160715160810982-565899129.png)

4. 打开example.sql ，执行上面的sql.这是测试用例。

5. 进入etc目录，拷贝配置文件， 

```
    cp  sphinx-min.conf.dist  sphinx.conf
```
  
(这里不拷贝sphinx.conf.dist ,因为这文件只是比前者多了一堆注释)

6. 打开sphinx.conf  修改下配置文件，如果使用的是步骤4的用例，那在这里只需要修改一下数据库配置就行。

![](http://images2015.cnblogs.com/blog/807718/201607/807718-20160715161243998-298675044.png)





7.  生成索引 /usr/local/sphinx/bin/indexer --all

8. 测试 /usr/local/sphinx/bin/search linux

 
测试可能会出错。所以不用管他。我们去安装coreseek吧！

### 安装coreseek

1. 首先下载软件，官网coreseek.cn网速奇慢。去百度云下载吧

    [https://pan.baidu.com/s/1c2u0fLI](https://pan.baidu.com/s/1c2u0fLI)

2. 解压oreseek里有2个文件夹 一个是mmseg中文分词包 还有一个是csft(其实就是sphinx)包 都要安装


首先安装mmseg中文分词

```
    ./configure --prefix=/usr/local/mmseg
```

编译时可能会报错

```
`config.status: error: cannot find input file: src/Makefile.in
```

通过automake来解决
首先检查是否安装了libtool如果没有 

```
yum -y install libtool
automake
```

如果automake报错 原因可能是下列

```
Libtool library used but `LIBTOOL' is undefined
The usual way to define `LIBTOOL' is to add `AC_PROG_LIBTOOL'
to `configure.ac' and run `aclocal' and `autoconf' again.
If `AC_PROG_LIBTOOL' is in `configure.ac', make sure
its definition is in aclocal's search path.
```

如果以上步骤都没成功，那么试下以下办法（把下面的命令都执行一遍，就好了）

``` 
    aclocal
    libtoolize --force
    automake --add-missing
    autoconf
    autoheader
    make clean
```


3. 然后继续mmseg的安装
```
    ./configure --prefix=/usr/local/mmseg
    make && make install
```

4. 安装csft

 ```
    ./configure --prefix=/usr/local/coreseek --with-mysql=/usr/local/mysql --with-mmseg=/usr/local/mmseg --with-mmseg-includes=/usr/local/mmseg/include/mmseg/ --with-mmseg-libs=/usr/local/mmseg/lib/
    make && make install

```

安装完毕后 注意 coreseek 中的配置文件也是`csft.conf` 而不是 `sphinx.conf`

```
cd /usr/local/coreseek/etc
cp sphinx.conf.dist csft.conf
vim csft.conf
```

5. 修改配置csft.conf，增加对中文支持

打开csft.conf

在index test1 段下面，增加

```
    charset_type = zh_cn.utf-8
    charset_dictpath =/usr/local/mmseg/etc/
```


6. 配置完成，测试一下吧

保存配置
建立索引
```
    cd /usr/local/coreseek/bin
   ./indexer --all
   ./search 哈喽
```


看看是不是已经出来效果了呢 （备注，原始测试的sql，都是英文，需要自己添加一些中文的记录哦）。


到这里就结束了，下一篇，将介绍下php怎么连接访问sphinx,coreseek