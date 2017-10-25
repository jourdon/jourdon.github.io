---
layout: post
title:  "使用Docker包laradock布署PHP开发环境"
categories: PHP
tags: Docker Laradock
author: 大东
---

* content
{:toc}


### 安装docker

安装这里就省略掉吧。配置这里需要注意一下，因为众所周知的原因，需要把 `Docker` 切换到国内镜像（我曾经因为没有配置镜像多次安装失败，心累！）
点击 `Docker-> Preferences->daemon->registry mirrors` ,这里可以添加国内镜像，可以使用网易或阿里云的镜像地址，这里不是重点。默认你添加完了，继续向下吧

### 安装Laradock
其它官方文档已经非常全面了，我只简单写下，主要是记录出错的解决方法

1 - 克隆项目到本地
``` 
    git clone https://github.com/Laradock/laradock.git

```
2 - 重命名 `env-example` 到 `.env` 。
``` 
    cd laradock 
    cp env-example .env
```
3 - 运行容器
``` 
    docker-compose up -d nginx mysql redis beanstalkd
```
4 -打开浏览器并访问本地主机：` http://localhost` 。
``` 
    That's it! enjoy :)
```






上面几个步骤其实已经把 `docker` 运行起来了，是不是相当的简单？但是我可以有一些其它的需求，比如，我现在的项目是比较老的项目，使用的是`TP3.1` 的框架，需要使用的 `PHP` 版本为 `5.4` ，需要同时使用 `mongo` 和 `mysql` 这两种数据库,这里就需要对 `laradock` 进行一些配置的更改了

### 重点在这里，配置laradock

我们先去查看一下 `.env` 文件，这里常规配置
``` 
    ############################
    # General Setup
    ############################
    
    ### Application Path
    # Point to your application code, will be available at `/var/www`.
    
    //APPLICATION=../ 这里是共享站点目录的位置,也就是当前的laradock目录的上级，为了方便管理，我在laradock的同级目录建了一个WWW目录做为网站目录，于是这里我改成下面了
    APPLICATION=../www/
    
    ### Data Path:
    # For all storage systems.
    //数据库地址。这里可以使用默认，
    DATA_SAVE_PATH=~/.laradock/data
    
    ### PHP version
    # Applies to the Workspace and PHP-FPM containers (Does not apply to HHVM)
    //这里是PHP默认支持的版本。只有三个选择，我只好选择5.6了
    # Accepted values: 71 - 70 - 56
    
    //PHP_VERSION=71  修改为
    PHP_VERSION=56
    
    ### PHP interpreter
    # Accepted values: hhvm - php-fpm
    
    PHP_INTERPRETER=php-fpm
```


继续向下就是一个 `PHP` 的扩展的配置，你需要开启什么扩展，就把他后面的 `false` 改成 `true` ,我需要使用 `mongo` , `mysql` 和 `redis` ,所以只需要改这个几，但是 `LDAP` 和 `V8JS` 也需要一起改，因为不改他们也会报错，这里不详细说了。

``` 
    ### WORKSPACE ##########################################################################################################
    
    WORKSPACE_INSTALL_XDEBUG=false
    WORKSPACE_INSTALL_LDAP=true
    WORKSPACE_INSTALL_SOAP=false
    WORKSPACE_INSTALL_MONGO=false
    WORKSPACE_INSTALL_PHPREDIS=true
    WORKSPACE_INSTALL_MSSQL=false
    WORKSPACE_INSTALL_NODE=false
    WORKSPACE_INSTALL_YARN=false
    WORKSPACE_INSTALL_DRUSH=false
    WORKSPACE_INSTALL_DRUPAL_CONSOLE=false
    WORKSPACE_INSTALL_AEROSPIKE=false
    WORKSPACE_INSTALL_V8JS=true
    WORKSPACE_COMPOSER_GLOBAL_INSTALL=false
    WORKSPACE_INSTALL_WORKSPACE_SSH=false
    WORKSPACE_INSTALL_LARAVEL_ENVOY=false
    WORKSPACE_INSTALL_LARAVEL_INSTALLER=false
    WORKSPACE_INSTALL_DEPLOYER=false
    WORKSPACE_INSTALL_LINUXBREW=false
    WORKSPACE_INSTALL_MC=false
    WORKSPACE_INSTALL_SYMFONY=false
    WORKSPACE_INSTALL_PYTHON=false
    WORKSPACE_INSTALL_IMAGE_OPTIMIZERS=false
    WORKSPACE_INSTALL_IMAGEMAGICK=false
    WORKSPACE_INSTALL_TERRAFORM=false
    WORKSPACE_INSTALL_DUSK_DEPS=false
    WORKSPACE_PUID=1000
    WORKSPACE_PGID=1000
    WORKSPACE_CHROME_DRIVER_VERSION=2.32
    WORKSPACE_NODE_VERSION=stable
    WORKSPACE_YARN_VERSION=latest
    WORKSPACE_TIMEZONE=UTC
    WORKSPACE_SSH_PORT=2222
    
    ### PHP_FPM ############################################################################################################
    
    PHP_FPM_INSTALL_XDEBUG=false
    PHP_FPM_INSTALL_MONGO=false
    PHP_FPM_INSTALL_MONGO_OLD=true
    PHP_FPM_INSTALL_MSSQL=false
    PHP_FPM_INSTALL_SOAP=false
    PHP_FPM_INSTALL_ZIP_ARCHIVE=false
    PHP_FPM_INSTALL_BCMATH=false
    PHP_FPM_INSTALL_PHPREDIS=true
    PHP_FPM_INSTALL_MEMCACHED=false
    PHP_FPM_INSTALL_OPCACHE=false
    PHP_FPM_INSTALL_EXIF=false
    PHP_FPM_INSTALL_AEROSPIKE=false
    PHP_FPM_INSTALL_MYSQLI=true
    PHP_FPM_INSTALL_POSTGRES=false
    PHP_FPM_INSTALL_TOKENIZER=false
    PHP_FPM_INSTALL_INTL=false
    PHP_FPM_INSTALL_GHOSTSCRIPT=false
    PHP_FPM_INSTALL_LDAP=true
    PHP_FPM_INSTALL_SWOOLE=false
    PHP_FPM_INSTALL_IMAGE_OPTIMIZERS=false
    PHP_FPM_INSTALL_IMAGEMAGICK=false
```
这里你会发现我的配置文件比你的配置多出了一行
``` 
    PHP_FPM_INSTALL_MONGO_OLD=true
```
没错，原本的配置文件里是没有这一行的，这个问题是因为`PHP5.6` 版本的问题，如果直接使用原本的 `PHP_FPM_INSTALL_MONGO=true` 这个配置，最后你会发现这样的报错 ，
```  
Error: Class 'MongoClient' not found
```
这个问题在 `laradock`  的 `issue` 里已经有人遇到过，[地址在这里](https://github.com/laradock/laradock/issues/254)，虽然官方没有修复，但是已经有人提出了解决方法，[地址在这里](https://github.com/laradock/laradock/issues/1112),好的，我们来看解决方法,需要更改以下三个文件。

`docker-compose.yml` 文件。
``` 
    INSTALL_BLACKFIRE=${INSTALL_BLACKFIRE}
    INSTALL_SOAP=${PHP_FPM_INSTALL_SOAP}
    INSTALL_MONGO=${PHP_FPM_INSTALL_MONGO}
    
    //多加了这里
    INSTALL_MONGO_OLD=${PHP_FPM_INSTALL_MONGO_OLD}
    
    INSTALL_MSSQL=${PHP_FPM_INSTALL_MSSQL}
    INSTALL_ZIP_ARCHIVE=${PHP_FPM_INSTALL_ZIP_ARCHIVE}
    INSTALL_BCMATH=${PHP_FPM_INSTALL_BCMATH}
```
`env-example` 文件，当然别忘记 `.env`文件。
``` 
   PHP_FPM_INSTALL_XDEBUG=false
   PHP_FPM_INSTALL_MONGO=false
   //这里多加了一行代码，别忘记改成true
   PHP_FPM_INSTALL_MONGO_OLD=false
   
   PHP_FPM_INSTALL_MSSQL=false
   PHP_FPM_INSTALL_SOAP=false
   PHP_FPM_INSTALL_ZIP_ARCHIVE=false  
```
`php-fpm/Dockerfile-56` 文件，后面的 `56` 对应是的 `PHP` 的版本号，在原本的 `MongoBD` 前面加入了 `Mongo(old)`.
``` 
     #####################################
     +# Mongo (old):
     +#####################################
     +
     +ARG INSTALL_MONGO_OLD=false
     +RUN if [ ${INSTALL_MONGO_OLD} = true ]; then \
     +    # Install the mongo extension
     +    pecl install mongo && \
     +    docker-php-ext-enable mongo \
     +;fi
     +
     +#####################################
      # MongoDB:
      #####################################
```
OK。。现在配置文件搞定了，去重新 `build` 一下吧,记得，你改过配置的都需要重新 `build` 。

``` 
    docker-composer build php-fpm workspace 
```
时间可能会久一些，抽根烟等一下吧。等到代码跑完就OK了，现在可以启动了

```
    docker-compose up -d nginx mongo mysql redis beanstalkd phpmyadmin
```

打开 `localhost` ,你又会看到熟悉的 `LARAVEL` 标志了。

HAPPY ENDING!