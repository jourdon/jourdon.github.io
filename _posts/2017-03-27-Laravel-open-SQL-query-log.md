---
layout: post
title:  "Laravel开启SQL语句查询日志"
categories: Laravel
tags: Laravel
author: 大东
---

* content
{:toc}

在实际开发中，我们需要查询当前SQL语句的状态，或者需要对SQL语句进行优化。开启查询日志，当然是很有必要的。

直接上代码吧
```
DB::connection('mysql')->enableQueryLog(); // 开启查询日志
```
中间是你的SQL语句代码
```
$queries = DB::connection('mysql')->getQueryLog(); // 获取查询日志
echo '<pre>';
var_dump($queries); // 即可查看执行的sql，传入的参数等等.
```