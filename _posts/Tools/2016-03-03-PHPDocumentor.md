---
title:  "PHPDocumentor"
date:   2016-3-3 12:14:54
layout: post
comments: true
categories: Tools
tags:  文档生成
---

* content
{:toc}

### 什么是PHPDocumentor？

`PHPDocumentor`是一个可以分析php源代码和注释块并生成文档的程序。目前最新的是`PHPDocumentor2`，基于`phpdocumentor1`和`javadoc`启发而来。支持`h5`语法以及`bootstrap`。可以生成的文档格式包括：PDF,HTML,CHM。

> 官网地址：http://www.phpdoc.org/

### PHPDocumentor的安装


**安装需求：**

- PHP 5.3.3或者更高
- [intl](http://php.net/manual/en/book.intl.php)扩展（提供国际化和字符编码支持）
- [Graphviz](http://www.graphviz.org/) 可选，用来生成类图


**intl扩展的安装：**

同其他php扩展一样，获取`php_intl`扩展后，在`php.ini`里面加载即可。

**Graphviz在windows下的安装：**

- 获取[安装文件](http://www.graphviz.org/pub/graphviz/stable/windows/graphviz-2.38.msi) ，注意：必须是`Graphviz.msi`,同安装exe文件一样。
- 使用pear安装`image_graphviz`，以便php可以操作`Graphviz`。命令：

```
pear install image_graphviz

downloading Image_GraphViz-1.3.0.tgz ...
Starting to download Image_GraphViz-1.3.0.tgz (16,878 bytes)
......done: 16,878 bytes
install ok: channel://pear.php.net/Image_GraphViz-1.3.0
```

- 修改通过pear安装的`imgae_graphviz`,路径，`php/pear/Image/Graphviz.php`：

```
var $dotCommand = 'D:/ruanjian/Graphviz2.38/bin/dot'; //Graphviz安装路径
var $neatoCommand = 'D:/ruanjian/Graphviz2.38/bin/neato';//Graphviz安装路径
```

- 写一个测试用例验证是否安装成功

**USAGE**

```
require 'd:/xampp/php/pear/Image/GraphViz.php';

$gv = new Image_GraphViz();
$gv->addEdge(array('商品详情'   => '添加购物车'));
$gv->addEdge(array('商品详情'   => '立即购买'));
$gv->addEdge(array('立即购买'   => '确认订单'));
$gv->addEdge(array('添加购物车' => '确认订单'));
$gv->addEdge(array('确认订单'   => '完成支付'));
$gv->image();
```

![image](/static/img/phpdoc.jpg)


**接下来开始安装`PHPDocumentor`**

**方式一：使用`PEAR`安装**

```
pear channel-discover pear.phpdoc.org
pear install phpdoc/phpDocumentor
```

**方式二：使用`PHAR`安装**

- 下载`PHAR`文件  http://www.phpdoc.org/phpDocumentor.phar
- 生成文档命令：

```
php phpDocumentor.phar -d . -t docs/api -o "HTML:frames:phpedit"     #-d 指定路径，-f指定文件，-t指定文档路径
```



**方式三：使用`composer`安装**

- 确保系统中已经安装好`composer`

- 在项目目录下创建文件`composer.json`，内容如下：

```
{
    "require-dev": {
        "phpdocumentor/phpdocumentor": "2.*"
    }
}
```

然后在此目录下使用命令： `composer install`

或者直接使用命令：`composer require "phpdocumentor/phpdocumentor:2.*"`，也会自动生成`composer.json`。



### PHPDocumentor 的使用

> 要想文档生成好，全靠`DocBlock`写的好

#### DocBlock

所有的文档性注释都是由 /** 开始的一个多行注释，在phpDocumentor里称为`DocBlock`。

 在 phpdocumentor 中，注释分为**文档性注释**和**非文档性注释**。phpdocumentor只识别用特定关键字标注的注释，比如author,var等，其他的注释不会被分析，也不会出现在最终生成的文档当中。
 
**一个DocBlock可能包含如下信息：**

- 功能简述区：文档性注释的第一行，对功能的简要概述
- 详细说明区：功能简述区后跟一空行，空行后可以对功能做更详细的概述，或者用法举例等，后面再跟一空行结束详细说明区
- 标记tag

**USAGE**

```
/**
 * 删除指定商品
 *
 * 根据goods_id删除商品，因此goods_id为必传参数，且不能为0
 *
 * @param int $goods_id 商品id
 * @return bool 
 */
function delGoods($goods_id)
{
    ...
}
```

![image](http://note.youdao.com/yws/public/resource/a4391ce0d947821f3ff072a6d98d861f/B5BCC95D21D0429EA93A518F32121EB0)


**支持的标记tag：**

标记 |使用范围 |描述
---|---|---
@access|class,function,var,define,module|用于指明关键字的存取权限：private、public或proteced
@author||指明作者
@copyright| class，function，var，define，module，use|指明版权信息
@deprecated|class，function，var，define，module，constent，global，include|指明不用或者废弃的关键字
@example||用于解析一段文件内容，并将他们高亮显示。会试图从该标记给的文件路径中读取文件内容
@const|define|指明php中define的常量
@final|class,function,var|指明关键字是一个最终的类、方法、属性，禁止派生、修改
@filesource||和example类似，只不过该标记将直接读取当前解析的php文件的内容并显示
@global||指明在此函数中引用的全局变量
@ingore||用于在文档中忽略指定的关键字
@license||相当于html标签中的<a>,首先是URL，接着是要显示的内容，例如<a href=”http://www.baidu.com”>百度</a>可以写作 @license http://www.baidu.com 百度
@link||类似于license但还可以通过link指到文档中的任何一个关键字
@name||为关键字指定一个别名
@package|页面级别的-> define，function，include，类级别的->class，var，methods|用于逻辑上将一个或几个关键字分到一组
@abstrcut||说明当前类是一个抽象类
@param||指明一个函数的参数
@return||指明一个方法或函数的返回
@static||指明关建字是静态的
@var||指明变量类型
@version||指明版本信息
@todo||指明应该改进或没有实现的地方
@throws||指明此函数可能抛出的错误异常,以及发生的情况


#### PHPDocumentor的一些注释规范

- 注释必须的形式:

```
/**
 * XXXXXXX
 */
```

- 对于引用了全局变量的函数，必须使用glboal标记
- 对于变量，必须用var标记其类型（int,string,bool...）
- 函数必须通过param和return标记指明其参数和返回值
- 对于出现两次或两次以上的关键字，要通过ingore忽略掉多余的，只保留一个即可
- 调用了其他函数或类的地方，要使用link或其他标记链接到相应的部分，便于文档的阅读。
- 必要的地方使用非文档性注释，提高代码易读性。
- 描述性内容尽量简明扼要，尽可能使用短语而非句子。
- 全局变量，静态变量和常量必须用相应标记说明



#### 生成文档

使用命令：`phpdoc.bat <-d|-f > 路径|文件 -t 文档存放路径`




#### 代码示例

```

/**
 * virtualShelves.php
 *
 * @author mayy
 * @time 2016-8-5
 * @version 1.420
 * @copyright (c) 2016, mayy
 */

/**
 * class VirtualShelves
 *
 * 测试使用phpdoc生成文档
 *
 * @author mayy
 * @time 2016-8-5
 */
class VirtualShelves{

    /** 
     * @var object $shopLogic shopLogic 
     */
    public $shopLogic = null;

    /**
     * 获取给门店代理店铺的商品
     *
     * @author mayy<myyd@outlook.com>
     * @time 2015-09-09 14:50
     * @param array $sstore_proxy_info 门店代理信息
     * @param array $params 查询参数
     * @param array $limit 分页
     * @return array
     */
    public function search_sstore_proxy_goods(array $sstore_proxy_info, array $params, array $limit = array())
    {
        return array(
            'count' => 3,
            'data' => array()
        );
    }

    /**
     * 获取给门店的推荐代理店铺,无分页的，一次推荐N个
     *
     * 1、获取全部满足条件的 推荐加权 的旗舰店id =>$recommend_store_ids
     * 2、获取全部满足条件的旗舰店id（不管是否有推荐加权）$all_store_ids
     * 3、从$recommend_store_ids中随机获取两个id
     * 4、从 $all_store_ids 中剔除第三步的两个id，然后再随机获取{N} - n 条id
     * 5、通过第四步得到的id获取对应的店铺数据
     *
     * @author mayy<myyd@outlook.com>
     * @time 2015-09-09 14:50
     * @param array $sstore_proxy_info 门店的代理信息
     * @return bool|array
     */
    public function get_recommend_stores_by_num(array $sstore_proxy_info)
    {
        return array();
    }
}
```

生成命令：

```
 phpdoc.bat  -d d:\server\apidoc\virtualShevles.php  -t d:\apidoc\
```












