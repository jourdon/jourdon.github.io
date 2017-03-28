---
layout: post
title:  "PHP抓取网页中的图片"
categories: PHP
tags: php函数
author: 大东
---

* content
{:toc}


爬取网页上的图片一般我们会用Python来爬取，但是PHP一样可以爬取图片哦，
直接上代码吧

```
$url="http://image.baidu.com/";
 
public function get_img($url)
{
   //获取网页中的图片在页面上显示出来
 //file_get_contents() 函数把整个文件读入一个字符串中
 $string=@file_get_contents($url);
 //preg_match_all函数进行全局正则表达式匹配。
 preg_match_all("/<img([^>]*)\s*src=('|\")([^'\"]+)('|\")/",$string,$matches);//带引号
 //preg_match_all("/<img([^>]*)\ssrc=([^\s>]+)/",$string,$matches);//不带引号
 //$matches[0]带有 <img class="st_camera_on" src=" 等标签
 //$matches[3]带有 直接获取SRC值
 $new_arr=array_unique($matches[3]);//去除数组中重复的值
 foreach($new_arr as $key){
  echo '<img src='.$key.'></br>';
 }
 
}
```

