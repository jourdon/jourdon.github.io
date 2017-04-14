---
layout: post
title:  "Laravel5.1 实现第三方登录认证（包括微博、QQ、微信、豆瓣）"
categories: Laravel
tags: Laravel OAuth
author: 大东
---

* content
{:toc}


第三方登录认证能简化用户登录/注册的操作，降低用户登录/注册的门槛，对提高应用的用户转化率很有帮助。

Socialite
------

`Laravel` 为我们提供了简单、易用的方式，使用 `Laravel Socialite` 进行 `OAuth`（OAuth1 和 OAuth2 都有支持） 认证。

`Socialite` 目前支持的认证有 `Facebook`、`Twitter`、`Google`、`LinkedIn`、`GitHub`、`Bitbucket`。（恩，有一半是“不存在”的网站,我们基本不会用到。）

`Socialite` 的用法官方文档中已经讲得很详细了，恕不赘述。

英文好的同学，建议直接看 Laravel 官方文档， 毕竟看二手知识是有高风险的 。

英文不好的同学（比如我），下面是中文文档：

Laravel 5.0. http://laravel-china.org/docs/5.0/authentication#social-authentication

Laravel 5.1. http://laravel.tw/docs/5.1/authentication#social-authentication

SocialiteProviders
------

`SocialiteProviders` 通过扩展 `Socialite` 的 `Driver`，实现了很多第三方认证。国内的有：`微博`、`QQ`、`微信`、`豆瓣`。当然你自己也可以参照实现其他的，只要那个网站支持 `OAuth`。







`SocialiteProviders` 的使用也超级简单易用，每个都对应了文档。其实，不懂英文也能看懂。

文档地址： http://socialiteproviders.github.io/

由于文档是基于 `Laravel5.0 的，我本身是使用Laravel5.1，那就说一下要注意的地方吧。

以 Weibo 为例
1. 安装
```
    composer require socialiteproviders/weibo
```

2. 添加 `Service Provider`
如果之前添加过 `Socialite Provider`，得先注释掉：

文件`config/app.php`
```
    'providers' => [
    //    Laravel\Socialite\SocialiteServiceProvider::class,
        SocialiteProviders\Manager\ServiceProvider::class, // add
    ],
```
3. 添加`Facades Aliase`
如果之前安装 `Socialite` 时添加过，就不需要再添加了。

还是文件`config/app.php`

```
    'aliases' => [
        'Socialite' => Laravel\Socialite\Facades\Socialite::class, // add
    ],
```
4. 添加事件处理器
文件`app/Providers/EventServiceProvider.php`
```
    protected $listen = [
        'SocialiteProviders\Manager\SocialiteWasCalled' => [
            'SocialiteProviders\Weibo\WeiboExtendSocialite@handle',
        ],
    ];
```
这里顺便提一下 `SocialiteProviders` 的原理。

`SocialiteProviders\Manager\ServiceProvider`实际上是继承于`Laravel\Socialite\SocialiteServiceProvider`的，这是它的源码：

```
    <?php
    
    namespace SocialiteProviders\Manager;
    
    use Illuminate\Contracts\Events\Dispatcher;
    use Laravel\Socialite\SocialiteServiceProvider;
    
    class ServiceProvider extends SocialiteServiceProvider
    {
        /**
         * @param Dispatcher         $event
         * @param SocialiteWasCalled $socialiteWasCalled
         */
        public function boot(Dispatcher $event, SocialiteWasCalled $socialiteWasCalled)
        {
            $event->fire($socialiteWasCalled);
        }
    }
```
它只是在启动时会触发`SocialiteWasCalled`事件，刚才在`SocialiteProviders\Manager\SocialiteWasCalled`事件的监听器中加上了事件处理器：`SocialiteProviders\Weibo\WeiboExtendSocialite@handle`。处理器的源码：

```
    <?php
    
    namespace SocialiteProviders\Weibo;
    
    use SocialiteProviders\Manager\SocialiteWasCalled;
    
    class WeiboExtendSocialite
    {
        public function handle(SocialiteWasCalled $socialiteWasCalled)
        {
            $socialiteWasCalled->extendSocialite('weibo', __NAMESPACE__.'\Provider');
        }
    }
```
处理器做的事情就是为 `Socialite`添加了一个 `weibo Driver`，这样就可以使用 `weibo` 的 `Driver` 了。

5. 添加路由
文件`app/Http/routes.php`

```
    // 引导用户到新浪微博的登录授权页面
    Route::get('auth/weibo', 'Auth\AuthController@weibo');
    // 用户授权后新浪微博回调的页面
    Route::get('auth/callback', 'Auth\AuthController@callback');
```

6. 配置
文件`config/services.php`

```
    'weibo' => [
        'client_id' => env('WEIBO_KEY'),
        'client_secret' => env('WEIBO_SECRET'),
        'redirect' => env('WEIBO_REDIRECT_URI'),  
    ],
```
文件`.env`

```
    WEIBO_KEY=yourkeyfortheservice
    WEIBO_SECRET=yoursecretfortheservice
    WEIBO_REDIRECT_URI=http://192.168.1.7/laravel/public/auth/callback
```
**注意**： `192.168.1.7` 是我本地虚拟机的地址，虚拟机可以连外网就可以测试了。貌似`QQ`的必须绑定域名才是测试。

当然，直接将配置的具体参数写在`config/services.php`中也是可以的，但是不推荐这样。因为`config/services.php`属于代码文件，而`.env`属于配置文件。当代码上线是只要应用线上环境的配置文件即可，而不需要改动代码文件，这算是一个最佳实践吧。

至于`WEIBO_KEY`和`WEIBO_SECRET`的具体值，这个是由新浪微博分发给你的，在新浪微博的`授权回调页`中填写`WEIBO_REDIRECT_URI`。这些细节已经超出本文的内容，建议直接到`http://open.weibo.com`查阅新浪微博的手册。

7. 代码实现
文件`app/Http/Controllers/Auth/AuthController.php`

```
	public function weibo() {
	return \Socialite::with('weibo')->redirect();
	// return \Socialite::with('weibo')->scopes(array('email'))->redirect();
}
    public function callback() {
        $oauthUser = \Socialite::with('weibo')->user();
        var_dump($oauthUser->getId());
        var_dump($oauthUser->getNickname());
        var_dump($oauthUser->getName());
        var_dump($oauthUser->getEmail());
        var_dump($oauthUser->getAvatar());
    }
```
访问`http://192.168.1.7/laravel/public/auth/weibo`，会跳转到新浪微博的`登录授权`页面，授权成功后，会跳转到`http://192.168.1.7/laravel/public/auth/callback`

返回的结果：

```
    string(10) "3221174302"
    string(11) "Mr_Jing1992"
    NULL
    NULL
    string(50) "http://tp3.sinaimg.cn/3221174302/180/40064692810/1"
```
`user`对象是现实了接口`Laravel\Socialite\Contracts\User`的，有以下几个方法：

```
    <?php
    namespace Laravel\Socialite\Contracts;
    interface User
    {
        public function getId();
        public function getNickname();
        public function getName();
        public function getEmail();
        public function getAvatar();
    }
```
当然，并不是有了这些方法就一定能获取到你需要的数据的。比如，在新浪的接口中，想要获取用户的`email`是得`用户授权`的，得到`授权`后请求获取邮箱的接口，才能拿到用户的邮箱。

详情参见：

`http://open.weibo.com/wiki/Scope`

`http://open.weibo.com/wiki/2/account/profile/email`

但是，`id`这个应该是所有第三方认证服务提供商都会返回的。不然那就没有办法作账号关联了。

获取到第三方的`id`后，如果这个`id`和你网站用户账号有绑定，就直接登录你网站用户的`账号`。如果没有任何账号与之绑定，就应该提示用户`绑定已有账号`或者是`注册新账号`什么的，这些具体逻辑就不在多说了。还有，在新浪上面还有一个 `取消授权回调页`的值需要填，是用户在`授权页`点击“取消”按钮时新浪回调的页面。这个可以设置为你网站的登录页面或者其他页面。

补充：

`http://socialiteproviders.github.io/providers/qq/` 文档中有一处错误。

```
    SocialiteProviders\QQ\QqExtendSocialite@handle
```
应该改为：
```
    SocialiteProviders\Qq\QqExtendSocialite@handle。 //注意大小写 。
```
