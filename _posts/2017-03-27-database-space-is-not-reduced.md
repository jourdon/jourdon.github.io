---
layout: post
title:  "mysql delete删除记录数据库空间不减少问题解决方法"
categories: Mysql
tags: Mysql
author: 大东
---

* content
{:toc}


记得在学习mysql时，delete删除记录只是给数据库中的记录加一个删除标识了，这样数据库空间并不是减少了，当时没想这么多，昨天发现一个数据库利用delete删除之后容量没变，后来百度了一下发现了下面一站长分享的文件，写得非常的不错，整理一下给各位参考。
 
我的linux虚拟机上只放了mysql数据库，今天发现数据库空间满了，检查了一下，发现数据表竟然占了5个多G。该删除释放一下空间了。果断delete之后发现数据库空间竟然没少，虽然数据记录数是零。

原来这是因为删除操作后在数据文件中留下碎片所致。DELETE只是将数据标识位删除，并没有整理数据文件，当插入新数据后，会再次使用这些被置为删除标识的记录空间。另外实际操作过程中还发现这个问题还存在两种情况。

（1）当DELETE后面跟条件的时候，则就会出现这个问题。如：
```
delete from table_name where 条件
```





删除数据后，数据表占用的空间大小不会变。

（2）不跟条件直接delete的时候。如：
```
delete from table_name
```
清除了数据，同时数据表的空间也会变为0。

这就存在了一个问题，在网站的实际运行过程中。经常会存在这样的附带条件删除数据的操作行为。天长日久，这不就在数据库中浪费了很多的空间吗。这个时候我们该使用 OPTIMIZE TABLE 指令对表进行优化了。

如何使用 OPTIMIZE 以及在什么时候该使用 OPTIMIZE 指令呢？
 
命令语法：`OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...`

最简单的：`optimize table phpernote_article;`
 
如果您已经删除了表的一大部分，或者如果您已经对含有可变长度行的表（含有VARCHAR, BLOB或TEXT列的表）进行了很多更改，则应使用 OPTIMIZE TABLE。被删除的记录被保持在链接清单中，后续的INSERT操作会重新使用旧的记录位置。您可以使用OPTIMIZE TABLE来重新 利用未使用的空间，并整理数据文件的碎片。
 
在多数的设置中，您根本不需要运行OPTIMIZE TABLE。即使您对可变长度的行进行了大量的更新，您也不需要经常运行，每周一次或每月一次即可，只对特定的表运行。
 
OPTIMIZE TABLE只对MyISAM, BDB和InnoDB表起作用。
 
注意，在OPTIMIZE TABLE运行过程中，MySQL会锁定表。因此，这个操作一定要在网站访问量较少的时间段进行。

```TRUNCATE```

其语法结构为：

```TRUNCATE [TABLE] tbl_name```

这里简单的给出个示例，

我想删除 friends 表中所有的记录，可以使用如下语句：

```truncate table friends;```

delete的效果有点像将mysql表中所有记录一条一条删除到删完，而truncate相当于保留mysql表的结构，重新创建了这个表，所有的状态都相当于新表,这样空间就减下来了。

好了，当然对于我们网站不可能使用truncate table来清除了，因这样之后所有数据都丢失了，这样肯定是不合理的清除了，我们必须使用delete来删除，然后再来修复优化表了哦。