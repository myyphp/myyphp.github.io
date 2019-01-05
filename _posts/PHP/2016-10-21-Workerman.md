---
layout: post
comments: true
categories: PHP
---



## Workman 简介

### 概念

`workerman` 是一个高性能的 `PHP socket` 服务器框架，workerman基于PHP多进程以及 `libevent` 事件轮询库，PHP开发者只要实现一两个接口，便可以开发出自己的网络应用，例如Rpc服务、聊天室服务器、手机游戏服务器等。

`workerman` 的目标是让PHP开发者更容易的开发出基于socket的高性能的应用服务，而不用去了解PHP socket以及PHP多进程细节。

`workerman` 本身是一个PHP多进程服务器框架，具有PHP进程管理以及socket通信的模块，所以不依赖php-fpm、nginx或者apache等这些容器便可以独立运行。

因此：

**`workerman` 是一个高性能的 `PHP socket` 服务器框架**

### 特点

- 纯PHP开发
- 支持PHP多进程
- 支持TCP、UDP
- 支持长连接
- 支持各种应用层协议
- 支持高并发
- 支持服务平滑重启
- 支持HHVM
- 支持以指定用户运行子进程
- 自带监控 ：`php start.php status`命令查看
- 支持毫秒级别定时器
- 支持异步IO
- 支持对象或者资源永久保持 ： 比如可以避免数据库的重复连接
- 高性能 ：php文件从磁盘读取解析一次后便会常驻内存，下次使用时直接使用内存中的opcode
- 诸多应用 ：Thrift-Rpc、Json-Rpc、 聊天室、统计监控服务、游戏后台等
- 支持分布式部署
- 支持心跳检测



## Linux下安装


1、环境检测： `curl -Ss http://www.workerman.net/check.php | php`,都ok后即可

如果提示php命令不找不到，就先创建命令的全局软链接 : `ln -s /usr/local/php/bin/php /sbin`

2、安装php的`libevent`扩展，如果还没有安装的话

```
wget http://pecl.php.net/get/libevent-0.1.0.tgz
tar -zxvf libevent-0.1.0.tgz
cd libevent-0.1.0/
phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install

vi  /usr/local/php/etc/php.ini
//加入一行
extension=libevent.so

service php-fpm restart
```

3、编写`start.php`代码，以及相关业务代码



## 运行模式

- 调试模式  ： `php start.php start` 

    > echo 等函数会把结果显示在终端，终端关闭，所有进程会退出

- `daemon` 运行模式 ： `php start.php start -d`         
    
    > 不会显示打印输出，可以通过 `Worker::$stdoutFile`属性 指定文件，这样输出的内容会记录到指定文件。终端关闭不受影响。


## 事件回调

- `onWorkerStart`
- `onConnect`
- `onMessage`
- `onError`
- `onWorkerReload`
- `onWorkerStop`
- `onClose`
- `onBufferFull` ： 发送缓冲区大小超过了 `TcpConnection::$maxSendBufferSize` 限制时触发
- `onBufferDrain` ： 在应用层发送缓冲区数据全部发送完毕后触发

## Webserver

### 运行机制

常驻内存，只要PHP文件被载入编译过一次，便会常驻内存，不会再去从磁盘读取或者再去编译。

### Webserver 使用注意

- 使用 `/Workerman/Protocols/Http.php` 文件中的`header`、`setcookie`、`sessionStart`等方法替换HTTP相关函数
- 无法使用 `move_uploaded_file()` `is_uploaded_file()` 这些函数


## 调试

- `php start.php start` ： 开启服务
- `php start.php stop`  ： 停止服务
- `php start.php status` ： 查看运行状态
- `strace -p 10354` ： 后面的数字为具体的pid数字，跟踪某进程做的事情



## 组件

- 进程间变量共享： `GlobalData`。(无法共享资源类型的数据)


- 进程间通讯或者服务器间通讯（多进程间） ： `Channel`

    原理图：
    
    ![image](/static/img/workerman.jpg)



## 使用注意

- 应用层协议不同
- 请求周期差异
- 注意避免类和常量的重复定义
- 注意单例模式的链接资源的释放
- 注意不要使用exit、die出语句
- 不要在主进程中初始化数据库、memcache、redis等连接资源
- 使用 `require_once` 加载文件


## 开发规范

- 类采用 **首字母大写** 的驼峰式命名，类文件名称必须与文件内部类名相同 ： `class MyProtocol`
- 使用命名空间，命名空间名字与目录路径对应，并以开发者的项目根目录为基准
- 普通函数及变量名采用小写加下划线方式 ： `get_connection_list()`
- 类成员及类的方法采用首字母小写的驼峰形式: `public $connectionList` 、 `public function getConnectionList()`
- 函数及类的参数采用小写加下划线方式 ： `function get_connect($one_param, $tow_param)`

