---
layout: post
title:  "Laravel5.1 多对多,实现用户与搜索词，文章和标签"
categories: Laravel
tags: Laravel 
author: 大东
---

* content
{:toc}


在实际的开发中，几种常见的对应关系模式：
```
    One-To-One //一对一
    One-To-Many //一对多
    Many-To-Many //多对多
```
举个例子：

- 一个用户对应一个用户中心

- 一个用户可以发表多篇文章

- 一篇文章可以有多个评论

而今天要要做的是多个用户搜索词的统计。
- 一个用户对应多个搜索词（一个人搜索了多少搜索词）
- 一个搜索词对应多个用户（多个用户搜索了同一个搜索词）

所有，用户和搜索词这前就是多对多的关系，这和文章与标签的多对多关系是相同的，比如，一篇文章可以有多个标签；一个标签可以属于多篇文章

借助`Laravel`的强大的`Eloquent`，实现这个功能还是比较顺心的。至于一对一和一对多这两种关系，可以触类旁通。

因为今天正常做了一个多对多关系，我就拿这个用户与搜索词做例子了

###创建表

要实现`Users-To-Keywords`这种`Many-To-Many`关系，我们需要`Users`表和`Keywords`模型，所以我们分别来创建。

`Laravel`自带有`artisan`命令和`migration`，可以很方便的建表，不过我个人比较习惯使用`Navicat`，所以我就使用`Navicat`来创建表了,

#### User表

| id       | name   |  password |
| -------- | -----: | :------:  |
| 1        | admin  |   admin   |
| 2        | guest  |   123456  |
| 3        | other  |   123456  |








#### Keyword表

| id       | name   |  created_at | updated-at |
| -------- | -----: |  ------:    |  :------:  |
| 1        | 好吃的  |2017-04-17 09:07:34|2017-04-17 09:07:34|
| 2        | 汉堡    |2017-04-15 19:15:08|2017-04-15 19:15:08|
| 3        | 好玩的  |2017-04-15 19:15:08|2017-04-15 19:15:08|


OK，生成这两个表之后，我们就可以正式开始我们的工作了。

### 声明`Eloquent`的关系

`Users`和`Keywords`是多对多的关系，所以我们需要在`UsersModel.php`中声明下面的关系：
```
    public function keywords()
        {
            return $this->belongsToMany('App\Models\Keyword');
        }
```
**`App\Models\Keyword`这个路径是因为我的`keyword`的模型路径是在`models`目录下，所以这里的路径要这么写，请参照下你自己的路径地址。**

在`Keyword.php`，也同样：
```
    public function users()
        {
            return $this->belongsToMany('App\Models\UsersModel');
        }   
```
我们使用`$this->belongsToMany()`来表明`Eloquent`的关系，这里需要注意的是如果你的外键并不是`user_id`和`keyword_id`，你需要在第三个参数进行设置，比如我的`user`表的`ID`并不是`user_id`,`Keyword`表的`ID`也并不是`keysord_id`,

```
    public function keywords()
        {
            return $this->belongsToMany('App\Models\Keyword','keyword_user','user_id','keyword_id')->withTimestamps();
            //第二个参数数据库里那个关联表名，第三个参数是本类在关联表中的字段，如果在当前表中和关联表中字段不同，这里需要传参，第四个参数是要查找的字段，如果在当前表中和关联表中字段不同，这里需要传参
            //如果你需要自动维护carated_at和updated_at，那么只需要在后面加上->withTimestamps()就可以了。
        }
```
OK，这样，我们的多对多关系就声明完毕了。

因为我们没用使用`artisan`命令以及`migration`和建立`User`表和`keyword`表，所以中间的关联表`keyword_user`(这里的关联表需要按字母顺序来绝定谁在前面)不会自动创建，这里需要我们手动建表。

#### keyword_user表

| keyword_id | user_id  | count| created_at | updated-at |
| -------- | -----: |  ------:    | ------:    |  :------:  |

正常情况下，count是不需要的，因为我要统计搜索次数，所以加了一个count字段


现在用户表和搜索词表、中间表都有了，我们开始写代码吧。

```
    //搜索词如果有返回实例，没有就写入
    $keyword=Keyword::firstOrCreate(['name'=>$name]);

    //找到当前用户的实例
    $user=UsersModel::find($uid);

    //如果没有登录，uid等于0
    if(!$user) $uid=0;

    //查找关联表中是否有对应的用户和搜索词
    $result=DB::table('keyword_user')->where('user_id',$uid)
        ->where('keyword_id',$keyword->id)
        ->first();

    if($result){
        //如果有就自增
        KeywordUser::where('user_id',$uid)
            ->where('keyword_id',$keyword->id)
            ->increment('count');
    }else{
        //没有就写入，
        if($user){
            //登录用户直接写入
            $user->keywords()->attach($keyword->id);
        }else{
            //未登录用户以用户ID = 0写入
            KeywordUser::firstOrCreate(['keyword_id'=>$keyword->id,'user_id'=>$uid]);
        }

    }
```
首先，查看keyword表中是否有当前搜索词，
这里使用的`firstOrCreata`，`firstOrCreata`会在数据库中查找，如果有就返回实例，没有就写入再返回实例。

第二步。查看`keyword_user`中是否已经存在搜索过的词。相同的用户，搜索的搜索词不能出现重复。所以，如果有存在的情况就让`count`自增，`count`是为了之后对搜索词进行统计,如果没有这个需求的话，这里就不需要写入了。直接跳过就好。

第三步，不存的情况，分为登录用户和临时用户

登录用户直接写入搜索词即可。这里使用的是`$user->keywords()->attach($keyword->id)`这里要注意`attach()`的用法

临时用户就需要另外的写入，因为`User`表中并没有临时用户，所以如果使用`attach()`来写入会出现错误，因为我们需要自动维护时间戳，所以这里需要使用`keyword_user`表的模型来写入，将临时用户的ID自定义为`0`.


### 查看用户的搜索词

存入了数据就会有查看数据，查看的代码如下

```
    //获取当前用户的搜索词
    $Users=UsersModel::find($uid);
    $data=$Users->keywords->toArray();

    foreach ($data as $key=>$val){
        $result[$key]['name']=$val['name'];
        $result[$key]['count']=DB::table('keyword_user')->where('user_id',$uid)
            ->where('keyword_id',$val['pivot']['keyword_id'])
            ->pluck('count');
    }
```
这里可以看出`$result`就是最后我们获取到的当前用户的搜索词数表,可以使用`dd($result)`来查看

OK.到这里我们完成了`Laravel`下的多对多关联。