---
title:  "ThinkPHP"
date:   2015-5-21 10:14:00
layout: post
comments: true
categories: Notes
tags: thinkphp
---

* content
{:toc}


# TP学习

> 本文记录了学习tp时的学习记录和一些知识点，便于以后查看、复习使用。




## 配置

**扩展配置**

扩展配置可以支持自动加载额外的自定义配置文件。如果在项目`Common/Conf/config.php`下配置则会到同级下寻找配置文件。也可以在模块下的`config.php`配置。

```php
// 加载Common/conf/下的user.php、db.php扩展配置文件
'LOAD_EXT_CONFIG' => 'user,db',  
```

**读取配置**

`C($config_name ,$value=null, $default)`  

- `$config_name` ：读取指定配置
- `$value` ：动态设置配置、修改
- `$default` ：指定默认值


读取二维配置：

`C('USER_CONFIG.USER_TYPE')`    


## Behavior

### 行为、标签位的理解

TP的行为可以理解为实现一些功能的操作。运行应用程序，实际是一个执行一些列**行为**的线性过程，我们可以在任意插入 **标签位** ，来让程序运行到此处时，先额外运行一下我们在此位置定义的行为方法代码。实际上就是一个钩子程序。


**实现过程：**

- 定义行为
- 配置标签位绑定的行为
- 标签位触发行为



### 定义行为

行为文件放的位置不重要，只要定义好命名空间，让程序能够加载到此文件即可，比如创建如下目录:`Common/Behavior/TestBehavior/TestBehavior.class.php`

```php
namespace Common\Behavior;
use \Think\Behavior;

class TestBehavior extends Behavior{
    //必须定义run方法，而且参数只有一个，因为是引用传递，因此必须是变量，多个参数可用数组
    public function run(&$param)
    {
	    echo $param;
    }
}
```

### 绑定行为

在`common/conf/tags.php`下定义行为绑定关系：

```php
return array(
    'app_init' => array(
		'Common\Behavior\MyAppInitBehavior'
	),
    'test' => array(
        'Common\Behavior\TestBehavior'
    ),
    //更多的标签位配置...
);
```

### 标签位触发行为

标签位分为：`系统标签位` 和 `自定义标签位`

系统核心提供的标签位置包括（按照执行顺序自动触发）：

- `app_init` 应用初始化标签位 
- `module_check` 模块检测标签位（3.2.1版本新增） 
- `path_info PATH_INFO` 检测标签位 
- `app_begin` 应用开始标签位 
- `action_name` 操作方法名标签位 
- `action_begin` 控制器开始标签位 
- `view_begin` 视图输出开始标签位 
- `view_template` 视图模板解析标签位 
- `view_parse` 视图解析标签位 
- `template_filter` 模板解析过滤标签位 
- `view_filter` 视图输出过滤标签位 
- `view_end` 视图输出结束标签位 
- `action_end` 控制器结束标签位 
- `app_end` 应用结束标签位 

自定义的标签位，需要在代码中使用`tag()`方法设置标签位后，才会触发

```php
public function index(){
    echo 1;
    $a = 'behavior';
    tag('test', $a);
    echo 2;
}
```

>应用程序在运行`index()`时候执行到`tag()`后先去执行 `TestBehavior` 类的 `run()`方法。然后再执行后续代码。





## 路由

> TP的路由功能 ： 可以通过定义一些规则，然后去匹配用户输入的 `url` 地址，匹配成功后，会去到对应的路径执行。


**开启路由：**

在应用配置或者模块配置文件中开启，并且当前`URL`模式不能是普通模式，且`URL`中的模块名不能被路由。

```php
//开启路由
'URL_ROUTER_ON'   => true, 
//规则定义
'URL_ROUTE_RULES'=>array(    
    'news/:year/:month/:day' => array('News/archive', 'status=1',array('method'=>'get')),),    //只有get请求才生效
    'news/:id'               => 'News/read',    
    'news/read/:id'          => '/news/:1',
),
```

TP的路由功能包括：

- 规则路由
- 正则路由
- 静态路由（URL映射）
- 闭包支持


### 规则路由

- `：`开头的参数为动态参数
- `\d` 会验证是否是数字
- 变量用`[ ]`包含起来后就表示该变量是路由匹配的可选变量,如：`[:month\d]`
- 非数字变量支持使用 `^` 进行简单的排除功能
- 可以使用 `$` 进行完全匹配

**USAGE**

```php
// www.shop.com/admin.php/test/2 可以被匹配
'test/:id\d' => 'index/index/test',

//.../index.php/Home/blog/2013 和 /index.php/Home/blog/2013/12都可以被匹配
'blog/:year\d/[:month\d]'=>'Blog/archive'

//匹配除了add edit 和delete之外的所有字符串
'news/:cate^add-edit-delete'=>'News/category'

//..index.php/Home/new/info/2不能被匹配
'new/:cate$'=> 'News/category'
```

### 正则路由

使用正则表达式，来匹配用户输入的url地址,必须要以 `/` 开头

**USAGE**

```php
'/^new\/(\d{4})\/(\d{2})$/' => 'News/achive?year=:1&month=:2',
```

### 静态路由

规则路由的简化版，不包含动态参数,且是`完全匹配`

**USAGE**

```php
//等同于 new/top$
'new/top' => 'news/index/type/top'
```

### 闭包支持

可以在路由规则里面定义一些闭包函数来处理一些需求，而不再执行控制器方法

**USAGE**

```php
'hello/:name' => function($name){
        echo 'Hello,'.$name;        
    }
```

### 路由规则冲突

有如下规则：

```php
'URL_ROUTER_ON'   => true,
'URL_ROUTE_RULES' => array(   
    'new/:id\d'    => 'News/read',
    'new/:name'    => 'News/read',
    'new/:year\d/:month\d'  => 'News/archive',
),
```

当访问URL：`http://serverName/index.php/Home/new/2012/03`时，希望第三条路由规则来匹配，但是实际上会被第一条规则匹配拦截。解决方法有：

- 调整规则顺序，把复杂的规则放在前面
- 使用完全匹配
- 利用正则路由



## 控制器

**前置和后置操作**

控制器方法支持前置和后置操作，执行顺序为：前置操作-当前操作-后置操作。一般只在当前操作里面做输出处理，前置、后置操作作用类似钩子。

**USAGE**

```php
public function index(){}
public function _before_index(){}
public function _after_index(){}
```

**参数绑定：**

```php
URL_PARAMS_BIND_TYPE'  => 0  //默认0：按变量绑定，1：按顺序绑定
```

**伪静态：**

使用伪静态更理由`SEO`优化。配置写法：

```php
'URL_HTML_SUFFIX'=>'shtml'                  //支持.shtml静态后缀
'URL_HTML_SUFFIX' => 'html|shtml|xml'       //支持多个静态后缀
'URL_HTML_SUFFIX' => ''                     //支持全部静态后缀
'URL_DENY_SUFFIX' => 'pdf|ico|png|gif|jpg', // URL禁止访问的后缀设置,优先级最高
```

配置`.html`的伪静态后访问：`http://serverName/Home/blog/3.html`，等同于：`http://serverName/Home/blog/3`


**U方法：生成URL**

`U('地址表达式',['参数'],['伪静态后缀'],['显示域名'])`

地址表达式：`[模块/控制器/操作#锚点@域名]?参数1=值1&参数2=值2...`

参数：支持字符串或者数组

```php
U('Index/test',array('id'=>3))
U('Index/test','id=3&cate=2')
U('/test/3') // 路由url，前提定义了规则，比如：'test/:id\d' => 'index/index/test' 
```

**页面跳转和重定向**

- `success('提示信息' [,跳转地址] [,跳转时间])` :默认跳转`$_SERVER["HTTP_REFERER"]`
- `error('提示信息' [,跳转地址] [,跳转时间])` : 默认跳转`javascript:history.back(-1)`
- `redirect('重定向地址' [,携带参数] [,跳转时间] [,跳转信息])` ： 地址、参数用法同`U`函数

如果想使用自定义的跳转模板页面，需要配置模板路径：

```php
//失败时的跳转模板页面会去找：view/public/error.html
'TMPL_ACTION_ERROR' =>'Public:error';
//成功后的跳转模板页面会去找：view/public/success.html
'TMPL_ACTION_SUCCESS' => 'Public:success';
```


**空控制器**

可以在控制器目录下定义一个`EmptyControllerClass`，当请求的控制器找不到时，会去执行控制器下的`index`方法。

**空操作**

可以在控制器里面定义一个`_empty()`方法，当请求的方法找不到时，会去执行该方法。



## 模型

> 使用`M()` 实例化模型性能比 `D()` 高，因为不需要加载具体的模型类。

**字段缓存**

`TP`会在模型首次实例化的时候自动获取数据表的字段信息，每个模型对应的缓存文件存放在`Runtime`目录下。在非调试模式下是默认开启缓存的，调试模式下默认关闭。该配置的修改：

```php
'DB_FIELDS_CACHE'=>true //线上由于数据表结构比较稳定，建议开启字段缓存
```

**`PDO`连接配置**

```php
//PDO连接方式
'DB_TYPE'   => 'pdo', // 数据库类型
'DB_USER'   => 'root', // 用户名
'DB_PWD'    => '', // 密码
'DB_PREFIX' => 'think_', // 数据库表前缀 
'DB_DSN'    => 'mysql:host=localhost;dbname=thinkphp;charset=UTF-8'
```

**连贯操作**

- `where` ：支持表达式的使用

```php
# 使用预处理查询字符串更安全
$user = I('user', '');
$user_info = M('admin')->where("username='%s'",array($user))->select(); # %s表示字符串
```

- `table` ：指定操作的数据表
- `alias` ：连表时取别名

```php
$Model = M('User');
$Model->alias('a')->join('__DEPT__ b ON b.user_id= a.id')->select();
```

- `data`  ：设置当前要操作的数据对象的值

```php
$Model = M('User');
$data['name'] = 'myy';
$data['password'] = md5(123123);
# 如果我们同时使用create方法和data创建数据对象的话，则最后调用的方法有效
$Model->data($data)->add();
```

- `field` ：标识要返回或者操作的字段

```php
$Model->field('id,SUM(score)')->select();    #可以使用函数
$Model->field(array('id','concat(name,'-',id)'=>'truename','LEFT(title,7)'=>'sub_title'))->select();
$Model->field(array('id','nickname'=>'name'))->select();    #数组方式还可以取别名
$Model->field('user_id,content',true)->select(); #字段排除
```

- `order`
- `limit`
- `page` ：用于分页，第一个参数为页数

```php
//page(1,20)相当于limit(0,20),page(2,20)相当于limit(20,20)
$user_info = M('admin')->page(1,20)->select();
```

- `group` ： 分组
- `having` ： 配合group方法完成从分组的结果中筛选（通常是聚合条件）数据

```php
$this->field('username,max(score)')->group('user_id')->having('count(test_time)>3')->select();
```

- `join` ： 联合查询，支持四个类型

```php
$res = M('admin')->alias('a')->join('left join sh_role as r on a.id = r.id')->select();
```

生成的`sql`语句：`SELECT * FROM sh_admin a left join sh_role as r on a.id = r.id`

- `union` ：合并两个或多个 `SELECT` 语句的结果集
- `distinct` ： 对结果进行去重

```php
$Model->distinct(true)->field('name')->select();
```

- `lock` ： 用于数据库的锁机制
- `cache` ： 缓存查询结果

```php
//单独制定缓存有效期和缓存类型,不制定的话，会使用配置项的 DATA_CACHE_TIME 和 DATA_CACHE_TYPE
$res = M('admin')->where("id=1")->cache(true, 60, 'Memcache')->find();
```

- `comment` ： 用于在生成的SQL语句中添加注释内容

```php
M('admin')->comment('查询第一条记录')->where("id=1")->find();
```

生成的`sql`语句：`SELECT * FROM `sh_admin` WHERE ( id=1 ) LIMIT 1   /* 查询第一条记录 */`


**AR模式**

tp实现了`ActiveRecords`模式的`ORM`模型，采用了非标准的`ORM`模型：表映射到类，记录映射到对象

```php
M('admin')->find(1);
M('admin')->select('1,3,8'); 
M('admin')->delete(8); 
```

**字段映射**

通过维护模型的 `protected $_map` 属性，该属性是数组格式的，表单字段和数据表真实字段的一个映射关系。从而达到在表单中隐藏真实字段名字的目的。


**表达式查询**

```php
$where['id'] = array('in', array(1,3,5));
//可以改写为:
$where['id'] = array('exp', 'IN (1,3,5)');

//通过 exp 更新数据
//实例化User对象
$User = M("admin"); 
//给要修改的数据对象属性赋值
$data['email'] = 'myyd@outlook.com';
$data['is_use'] = array('exp', 'is_use+1');
$model->where('id=1')->save($data);
```
生成的`sql`语句：

```php
UPDATE `sh_admin` SET `email`='myyd@outlook.com',`is_use`=is_use+1 WHERE ( id=1 )
```

**查询语言**

- 修改多个条件之间的逻辑关系

在使用 `where($param)` 时数组`$param`条件的元素之间的各关系默认是 `and` 的关系，如果想修改成 `or` 关系的话，可以如下使用：

```php
$param['name'] = 'may';
$param['age'] = 21;
$param['_logic'] = 'OR';
M('admin')->where($param)->select();
```

- 对同一个字段进行多条件筛选

如果想对某个字段进行区间范围筛选，比如：

```php
//查找 id<10 or id>30的记录
$where['id'] = array(array('lt',10), array('gt',30), 'or');
//查找 10<id<30的记录
$where['id'] = array(array('lt',30), array('gt',10), 'and');
```

- 组合查询

字符串条件：

```php
//字符串条件和数组条件一起舒勇
$where['name'] = 'mayy';
$where['_string'] = 'is_use=1 AND id>1';
M('admin')->where($where)->select(); 
```

复合查询：把前面的条件通过 `_complex` 组装在一起，再和其他条件 `and` 在一起，以完成更复杂的查询

```php
$where['name']  = array('like', '%mayy%');
$where['age']  = array('gt',23);
$where['_logic'] = 'or';
$params['_complex'] = $where;    
$params['id']  = array('gt',1);
M('admin')->where($params)->select();
```

- 子查询

```php
//1.构建需要的子查询sql语句
$subQuery = $model->where($where)->buildSql();
//2.利用子查询进行查询 
$model->table($subQuery.' a')->where($where)->order($order)->select() 
```

**自动验证**

可以在使用create创建数据对象的时候自动进行数据验证，有两种方式：

- 静态验证 ：通过模型类的`$_validate`属性定义验证规则
- 动态验证 ：通过模型类的`validate()`方法动态创建自动验证规则

验证规则格式：

```php
array(     
    array(验证字段1,验证规则,错误提示,[验证条件,附加规则,验证时间]),
    array(验证字段2,验证规则,错误提示,[验证条件,附加规则,验证时间]),     
    ......
);
```

**自动完成**

如果配置了自动完成，在使用`create()`的时候会完成对数据自动处理和过滤的方法，比如可以配置对密码字段自动使用`md5()`加密函数处理。同自动验证类似，也是有两种方式

- 静态验证 ：通过模型类的`$_auto`属性定义处理规则
- 动态验证 ：通过模型类的`auto()`方法动态创建自动处理规则


**参数绑定**

需要`PDO`的支持，利用`bind()方法`，来预处理的sql语句中的变量占位符进行绑定。



## 视图

默认的模板文件定义规则： 

`视图目录/[模板主题/]控制器名/操作名+模板后缀`

- 配置整个视图目录所在位置 ： `'VIEW_PATH'=>'./Tpl/'`
- 配置视图目录名字：`'DEFAULT_V_LAYER'  =>  'View'`
- 配置模板后缀： `'TMPL_TEMPLATE_SUFFIX'=>'.html'`
- 配置目录层级： `'TMPL_FILE_DEPR'=>'_'` （控制器名/模板文件 会变成 控制器名_模板文件）
- 配置模板主题： `'DEFAULT_THEME' => 'default'` ，会在控制器前多增加一层级目录来区别主题


模板使用相关方法：

- `assign('name', $val)`    ： 分配变量
- `display('[模板文件]'[,'字符编码'][,'输出类型'])` ： 加载模板并输出
- `fetch('模板文件')` ： 加载模板内容
- `show('渲染内容'[,'字符编码'][,'输出类型'])` ： 直接把指定内容通过`display`输出出来


## 模板

> 模板在执行过程中都会在`Runtime/模块/Cache/`下生产一个编译后的php缓存文件

tp内置的模板引擎支持两种标签定义：

- 普通标签
- `XML`标签


配置普通标签的标记：

```php
'TMPL_L_DELIM'=>'{',   #开始标签
'TMPL_R_DELIM'=>'}',   #结束标签
```


**USAGE**

```php
//自定义变量需要先分配
{$name}                   //在模板中输出普通变量
{$user.email}             //输出数组里面的一个元素
{$user['email']}          //等价于上面，建议此种方式
{$user->name}             //输出对象的一个属性

//系统变量直接使用
{$Think.server.script_name}   // 等价于输出$_SERVER['SCRIPT_NAME']
{$Think.session.user_id}      // 等价于输出$_SESSION['user_id']
{$Think.get.pageNumber}       // 等价于输出$_GET['pageNumber']
{$Think.cookie.name}          // 等价于输出$_COOKIE['name']

//使用函数
{$name|md5}                   //对name变量进行md5函数处理
{$date|date="y-m-d",###}      //date函数需要两个参数，变量非第一个位置时用###标识
{$name|substr=0,3}            //name变量位于第一个参数位置
{$name|md5|strtoupper|substr=0,3} //多个函数用|隔开，顺序从左至右
{$name|myfun}                 //使用自定义函数，比如我在functions.php里面定义了myfun()

//使用默认值
{$email|default='daf@qq.com'} //当$email没有值是，会使用默认值输出

//使用运算符
{$store['score'] + 10}        //使用运算符时，不能用点语法和用|的方式使用函数

//文件包含
<include file="Public/header" />  //包含头部模版header
<include file="Public/header" title='传递的标题'/>  //包含文件并传递一个参数,在被包含文件中使用的话: [title]

//内置标签
//volist：遍历数组。属性较多
<volist name="data" id="vo" empty="$empty" key="k">
    {$vo.id}:{$vo.username}
    <br/>
</volist>

//foreach：遍历数组，使用简单
<foreach name="list" item="vo" >
    {$key}|{$vo.id}:{$vo.name}
</foreach>

//for：循环
<for start="开始值" end="结束值" comparison="" step="步进值" name="循环变量名" >
    {$i}
</for>

//switch
<switch name="变量" >
    <case value="值1" break="0或1">输出内容1</case>
    <case value="值2">输出内容2</case>
    <default />默认情况
</switch>

//比较判断:neq、gt、egt、lt、elt、heq、nheq
<eq name="Think.get.name" value="value">相等
<else/>不相等
</eq>

//范围判断:in notin between notbetween
<in name="id" value="$range">id在范围内
<else/>id不在范围内
</in>

//if标签
<if condition="strtoupper($user['name']) neq 'ADMIN'">admin
<else /> zhangsan
</if>

//empty：判断变量是否为空
<empty name="Think.get.name">$_GET['name']为空值
<else /> $_GET['name']不为空值
</empty>

//present ：判断变量是否已经定义
<present name="name">name已经赋值
<else /> name还没有赋值
</present> 

//defined ：判断常量是否已经定义
<defined name="NAME">NAME常量已经定义
<else /> NAME常量未定义
</defined> 

//assign ：在模板中给变量赋值
<assign name="var" value="$Think.get.name" />

//导入外部js、css ：import、load、js、css
<import type="css" file="admin.css.public"/>   //路径为命名空间方式
<load href="__PUBLIC__/Admin/css/public.css" />   //文件真实路径

//literal ：防止模板标签被解析
<php><literal>echo '{$Think.config.CUSTOM.'.$key.'}';</literal></php>

//模板注释
{// 这是单行模板注释内容 } 

{/* 这是多行模板
注释内容*/ }
```


**模板布局**

TP的模板布局原理：

通过创建一个模板文件来布局，比如：

```php
<include file="Public:header" />
{__CONTENT__}
<include file="Public:footer" />
```
然后，在加载其他模板文件的内容替换到`{__CONTENT__}`中，实现相同布局时候的重复利用。

TP内置的模板引擎支持三种布局方式：

- 全局配置方式 ：需要配置开启布局和指定布局文件

```php
'LAYOUT_ON'=>true,  
'LAYOUT_NAME'=>'layout', //每次display时会去找布局文件：layout.html
'TMPL_LAYOUT_ITEM' =>  '{__REPLACE__}',   //可以修改{__CONTENT__}
```

- 模板标签方式 ： 不需要进行任何设置，在模板文件的头部加入`layout`标签表明是否需要使用模板布局

```php
<layout name="layoutname"  replace="{__REPLACE__}" />     //replace可以不写，用默认的
```

- `layout()`方法控制方式 ： 在控制器里面通过`layout(bool)`来灵活控制当前要加载的模板是否使用布局

```php
Public function add() {
    layout(true);       //此时不需要配置开启也可以使用     
    $this->display('add');     
}
```


**模板继承**

TP支持一种模板继承的布局方式，其原理是：

通过定义一个基础模板，在基础模板里通过`<block></block>`规划好布局。然后在子模板中使用`<extend name="路径" />`后，对基础模板规划好的`block`块通过对应的`name`属性，进行重写，然后就会把重写后的内容放到基础模板对应的`block`中去展示。


**模板替换**

TP在加载模板文件后，会对一些特殊的字符串（区分大小写）进行替换。默认的包括：

-  `__ROOT__` ： 会替换成当前网站的地址（不含域名） 
-  `__APP__` ： 会替换成当前应用的URL地址（不含域名）
-  `__MODULE__` ：会替换成当前模块的URL地址 （不含域名）
-  `__CONTROLLER__` ：（或者 `__URL__`兼容考虑）会替换成当前控制器的URL地址（不含域名）
-  `__ACTION__` ：会替换成当前操作的URL地址 （不含域名）
-  `__SELF__` ： 会替换成当前的页面URL
-  `__PUBLIC__` ：会被替换成当前网站的公共目录 通常是 `/Public/`


除了默认的之外，也可以自定义一些想被替换的字符串：

```php
'TMPL_PARSE_STRING'  =>array(
    '__PUBLIC__' => '/Common',       // 更改默认的/Public 替换规则     
    '__JS__'     => '/Public/JS/',   // 增加新的JS类库路径替换规则
    '__CSS__'     => '/Public/CS/',  // 增加新的css类库路径替换规则
    '__UPLOAD__' => '/Uploads',      // 增加新的上传路径替换规则
    ),    
```



## 调试

调试模式的好处：

- 开启日志记录 
- 关闭模板缓存
- 记录SQL日志
- 关闭数据表字段缓存
- 严格检查文件大小写 
- 通过页面Trace功能更好的调试和发现错误

开启调试模式只需要在入口文件处定义即可：

```php
define('APP_DEBUG', true);
```

关闭调试模式后，发生的错误信息不会具体提示，如果仍然希望看到，可以配置：

```
'SHOW_ERROR_MSG' =>  true,    // 显示错误信息
```

**日志记录**

非调试模式下，如果要开启日志记录，需要配置 ：

```php
'LOG_RECORD' => true,                      // 开启日志记录
'LOG_LEVEL'  =>'EMERG,ALERT,CRIT,ERR',     //只记录EMERG ALERT CRIT ERR 错误
'LOG_TYPE'   =>  'File',                   // 日志记录类型 默认为文件方式
```

如上配置，代码在运行时出现了配置错误级别的错误时，会自动记录日志，但是有时候业务需求更多的需要我们手动记录一些日志来更方便的调试，此时可以使用 `\Think::Log` 类的方法 ：

方法 | 作用
--- | ---
`record('信息', '错误级别', '是否强制记录')` | 记录日志信息到内存，在请求结束后，会调用`save()`方法
`save()` | 把保存在内存中的日志信息（用指定的记录方式）写入
`write()` | 实时写入一条日志信息


**页面trace**

开启页面`trace`可以查看到更多的调试信息

```php
'SHOW_PAGE_TRACE' =>true,   //开启页面trace
```

![img](/static/img/tp1.jpg)


**断点调试**

只需要在不同的位置对某个变量进行trace输出即可

```php
$res = array();
trace($res);
$res = M('admin')->select();
trace($res);
```

结果：

![img](/static/img/tp2.jpg)

**性能调试**

```php
G('begin'); 
// ...其他代码段
G('end');
// 进行统计区间
echo G('begin','end').'s';          //时间统计
echo G('begin','end','m').'kb';     //内存占用统计
```

**错误调试**

```php
E($msg);    //输出错误信息，并中断执行。类似真实报错的页面。而非单纯的输出信息。
```

**模型调试**

```php
echo M('admin')->getLastSql();
echo M('admin')->_sql();         //3.2版本的简化方法
echo M('admin')->getDbError();   //如果CURD操作返回false，可以用此获取数据库的错误信息
```

## 缓存

TP提供的缓存包括：

- 数据缓存 ： 利用 `S()`方法
- 快速缓存 ： 利用 `F()`方法
- 查询缓存 ： 利用 `cache()` 方法
- SQL解析缓存 ： 需要配置是否开启以及缓存方式

```php
'DB_SQL_BUILD_CACHE' => true,
'DB_SQL_BUILD_QUEUE' => 'file',   //默认为file,还可以支持xcache和apc
'DB_SQL_BUILD_LENGTH' => 20,      // SQL缓存的队列长度
```

- 静态缓存 ： 需要配置开启以及缓存规则,生成的文件在`HTML_PATH`定义的目录下，且只有`get`请求才会生效

```php
'HTML_CACHE_ON'     =>    true, // 开启静态缓存
'HTML_CACHE_TIME'   =>    60,   // 全局静态缓存有效期（秒）
'HTML_FILE_SUFFIX'  =>    '.shtml', // 设置静态缓存文件后缀
'HTML_CACHE_RULES'  =>     array(  // 定义静态缓存规则     
    // 定义格式1 数组方式     
    '静态地址'    =>     array('静态规则', '有效期', '附加规则'),     
    // 定义格式2 字符串方式     
    '静态地址'    =>     '静态规则', 
)
```

静态地址：要进行静态缓存的操作，比如 `index:index`，表示`index`控制器的`index`方法

静态规则：用于定义要生成的静态文件的名称，要确保名字不会冲突，比如 `{:module}/{:controller}_{:action}_{id}` ，该规则就会在静态目录下创建模块文件夹，然后创建控`制器名_方法名_$_GET['id']`的静态文件



## 安全

`web`安全注意建议 ：

- 对外来数据做安全验证
- 对所有公共的操作方法做必要的安全检查，防止用户通过URL直接调用
- 不要缓存需要用户认证的页面
- 对用户的上传文件，做必要的安全检查，例如上传路径和非法格式
- 如非必要，不要开启服务器的目录浏览权限
- 对于项目进行充分的测试
- 做好服务器的安全防护



**表单合法性检测**

- 配置 `insertFields` 和 `updateFields` 属性来限制更新和插入    
- 使用 `field()` 方法动态限制


**表单令牌**

1. 配置牌验证

```php
'TOKEN_ON'      =>    true,  // 是否开启令牌验证 默认关闭
'TOKEN_NAME'    =>    '__hash__',  // 令牌验证的表单隐藏字段名称，默认为__hash__
'TOKEN_TYPE'    =>    'md5',  //令牌哈希验证规则 默认为MD5
'TOKEN_RESET'   =>    true,  //令牌验证出错后是否重置令牌 默认为true
```

2. 配置标签位`view_filter=>array('Behavior\TokenBuild'),`

3. 如果希望指定页面不进行令牌验证，可以在控制器 `display()` 前关闭验证:`C('TOKEN_ON',false);`

4. 在使用 `create()` 时会自动验证令牌，如果没有使用`create()`，可以手动调用模型方法`autoCheckToken()`


**防止SQL注入**

- 查询条件尽量使用数组方式
- 字符串条件，使用预处理机制
- 使用自动验证和自动完成进行自定义的过滤
- 使用`PDO`进行参数绑定

**模板文件安全**

- 可以配置`.htaccess`文件限制模板文件目录服务权限

**防止XSS攻击**

- 过滤所有的`JS`脚本
- 使用`htmlspecialchars`等函数转义`html`元字符
- 使用tp的`remove_xss`方法













