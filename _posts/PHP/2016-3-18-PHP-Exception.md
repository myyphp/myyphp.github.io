---
layout: post
comments: true
categories: PHP
---

# 异常处理

## 简单介绍

异常处理是一种面向对象思想的错误处理机制。

一个简单的异常处理如下：

```
try{
    throw new Exception('对方不想和你说话，并抛出了一个异常');
} catch(Exception $e) {
    echo $e->getMessage();
}
```

传统的错误处理，往往依赖于函数的返回值，然后对返回值进行判断处理，因此这样的判断处理地方可能会存在很多。而使用异常处理的话，只用在业务代码里面可能出问题的地方抛出相应的异常，然后就可以在代码入口处对异常做统一处理，这样的代码往往看起来更整洁、规范、易于管理。

## 结构说明

- `throw` ： 抛出一个指定类型的异常
- `try`   ： 检测包住的代码，执行中有无异常抛出，若有，就终止后续代码执行，而去执行`catch`代码段
- `catch` ： 捕获指定类型的异常，捕获到就执行相应的代码段，可以针对不同类型的异常写多个`catch`，如果全部类型都捕获不了抛出的异常类型，则会发生一个`Fatal error`


## PHP内置异常

上面例子中的代码 : `throw new Exception()`，实例化的实际上就是PHP内置的异常处理类,结构中，可以被外部调用的方法如下 ：

```
class Exception  
{  
    function __construct($message = null, $code = 0);  
    //异常信息
    final function getMessage();
    //异常错误码即：$code
    final function getCode(); 
    //发生异常的文件名
    final function getFile();   
    //异常的行号
    final function getLine();
    //backtrace()得到的数组 
    final function getTrace();        
    //把backtrace()结果数组转成字符串显示
    final function getTraceAsString();
    //异常信息字符串化，可以重写并自定义展示规则
    function __toString();
} 
```


## 自定义异常处理类

往往在业务开发中，内置的`Exception`类无法满足我们的需求，这时可以通过定义一个异常类并继承自`Exception`类，然后就可以在自定义异常类里面做很多自定义处理。

比如现在有个需求，希望在调用某个方法出现问题时，给用户更友好的提示，比如一个定制的500页面。

- 我们便可以定义一个异常类，并定义一个用于展示500页面的方法

```
class MyException extends Exception{
    public function page500(){
    	//这里可以加载500页面模板...
    	exit('here is 500 page');
    }
} 
```

- 在业务代码里面进行相应的异常抛出、处理

```
try {  
    // some codes... 
    throw new MyException('自定义异常');  
} catch (MyException $e) {
    $e->page500();
} catch (Exception $e) {
    //异常基类可以用于捕获更多的异常类型
    echo $e->getMessage();
}
```

## 顶层异常处理

如果代码里面抛出了异常，而又没有被捕获，或者捕获的类型匹配不上，还是会报出一个`Fatal error`，为了防止这种不友好的错误展示给用户，系统函数 `set_exception_handler()` 闪亮登场，该函数可以用来捕获所有未被捕获处理的异常类型。

```
//异常处理定制
function exception_handler($e) 
{ 
    //$e是一个异常对象 
    print $e;
}

//顶层异常捕获处理
set_exception_handler("exception_handler"); 
```

如上注册异常处理后，就可以防止`Fatal error`的发生。