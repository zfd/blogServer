---
title: PHP核心技术与最佳实践
date: 2018-03-01 10:02:32
categories: 笔记
tags: [php, book note]
---

## 面向对象的的核心概念

### 面向对象

> `面向对象`（OO）：是软件开发方法。
  `面向对象编程`（OOP）：将对象作为程序的基本单元，把程序和数据封装其中，以提高软件的重用性、灵活性和可拓展性。

### 序列化、反序列化

> 对象的底层实现：“属性数组”+“方法数组”。
  对象的序列化：把保存在内存中的对象的属性保存起来，并且可以在需要的时候还原出来。（serialize）

### 继承、封装、多态

> 面向对象思想的`三大要素`：继承、封装、多态。
  `作用`：让代码更具开放性、可扩充性，增加代码的重用性、提高软件的可维护性。

#### 继承

作用：继承一个公共类，可以使用一些公有方法，提升程序的效率，减少代码的重复。

+ 用关键字extends实现继承。
+ 子类只能继承父类的非私有属性。
+ 子类继承父类后，相当于将父类的属性和方法copy到子类，可以直接使用$this调用该属性。
+ PHP只能单继承，不支持一个类继承多个类。但是一个类可以进行多层继承（即A继承于B，而C又继承于A，C通过A间接继承了B）。 

#### 封装

作用：防止代码冗余，也可以方便代码的调用。

#### 多态

概念：同一个类的不同子类表现出不同的形态。（子类重写父类的方法）

### 反射

> 通过反射，可以找到一个对象所属的类及拥有的方法。
> 在平常开发中，用到反射的地方不多：一个是对对象进行调试，另一个是获取类的信息

```
<?php
class person {
    public $name;
    public $gender;

    public function say() {
        echo $this->name, " ", $this->gender, "\r\n";
    }

    public function set($name, $value) {
        echo "Setting $name to $value \r\n";
        $this->$name = $value;
    }

    public function get($name) {
        if (!isset($this->$name)) {
            echo '未设置';
            $this->$name = "正在为你设置默认值";
        }
        return $this->$name;
    }
}

$student = new person();
$student->name = 'Tom';
$student->gender = 'male';
$student->age = 24;

// 获取对象属性列表
$reflect = new ReflectionObject($student);
$props = $reflect->getProperties();
foreach ($props as $prop) {
    print $prop->getName() . "\n";
}

// 获取对象方法列表
$m = $reflect->getMethods();
foreach ($m as $prop) {
    print $prop->getName() . "\n";
}

// 返回对象属性的关联数组
var_dump(get_object_vars($student));
// 类属性
var_dump(get_class_vars(get_class($student)));
// 返回由类的方法名组成的数组
var_dump(get_class_methods(get_class($student)));
// 获取对象属性列表所属的类
echo get_class($student);
```

### 异常

+ Deprecated最低级别错误，程序继续执行
+ Notice 通知级别的错误 如直接使用未声明变量，程序继续执行
+ Warning 警告级别的错误，可能得不到想要的结果
+ Fatal error  致命级别错误致命级别错误，程序不往下执行
+ parse error 语法解析错误，最高级别错误，连其他错误信息也不呈现出来
+ E_USER_相关错误 用户设置的相关错误

error_reporting(-1)显示所有错误，error_reporting(0)屏蔽所有错误。

error_reporting(E_ALL&~E_NOTICE)不显示通知级别的错误。“~”表示非。

## PHP网络技术及应用

### Web 基础
HTTP（HyperText Transfer Protocol，超文本传输协议）。
WWW（World Wide Web）的三种技术：HTML、HTTP、URL。
RFC（Request for Comments，征求修正意见书），互联网的设计文档。

URI（Uniform Resource Indentifier，统一资源标识符）
URL（Uniform Resource Locator，统一资源定位符）
URN（Uniform Resource Name，统一资源名称），例如 urn:isbn:0-486-27557-4
URI 包含 URL 和 URN，目前 WEB 只有 URL 比较流行，所以见到的基本都是 URL。

### HTTP请求

> CRLF：Carriage-Return Line-Feed，即回车+换行，\r\n

+ 请求行：请求方法 + URL + 协议/版本 + CRLF，之间由空格分隔 
+ 请求头：key + ":" + value + CRLF （N行）
+ 空行
+ 请求正文：Get请求没有正文/Post请求正文是参数

### HTTP响应

+ 状态行：协议/版本 + 状态码 + 状态码描述 + CRLF，之间由空格分隔
+ 响应头：key + ":" + value + CRLF （N行）
+ 空行
+ 响应正文：html/json等

响应消息：
1xx:信息响应类，表示接收到请求并且继续处理
2xx:处理成功响应类，表示动作被成功接收、理解和接受
3xx:重定向响应类，为了完成指定的动作，必须接受进一步处理
4xx:客户端错误，客户请求包含语法错误或者是不能正确执行
5xx:服务端错误，服务器不能正确执行一个正确的请求

## PHP模板引擎的原理和实践

### smarty（直接看smarty就可以了）

> smarty是基于PHP开发的PHP模板引擎，他提供了php逻辑与html页面的分离

工作流程：

+ 把需要显示的变量，赋值，塞到对象的内部属性中的一个数组里 
+ 然后编译模板，将标签解析成相应的php echo代码 
+ 引入编译后的php文件

使用步骤：

+ Smarty是一个类，要使用的话，必须引入在进行实例化 
+ 使用assign给模板赋值 
+ 使用display方法（从编译到输出）

优点：

+ 速度快：相对于其他模板引擎
+ 编译型：把模板文件替换成一个HTML+PHP混合的PHP文件，当下模板没有改变，将自动转向编译文件
+ 缓存技术：一定缓存时间内，用户最终看到的html文件缓存成一个静态的html页
+ 插件技术：可以自定义插件

缺点：

+ 编译模板，浪费时间 
+ 要把变量再重新赋值到对象的属性中，增大了开销
+ 不适用用实时更新，股票、天气等
+ 不适用于小项目，小项目直接开发更快

Smarty.class.php：

```
<?php
 
class Smarty  //此类就是libs中的Smarty.class.php类
{
    public $leftlimit="<{";  //左分隔符
    public $rightlimit="}>";//右分隔符
    public $attr;  //存放变量信息的数组
     
     
    //注册变量
    function assign($k,$v)
    {
        $this->attr[$k] = $v;  //向数组中添加一个值,相当于$sttr[0]="sdc123"
    }
     
    //显示模板
    function display($name)
    {
        //1.造模板路径
        $filename = $mubanlujing.$name;
         
        //2.获取模板内容,内容是一大串代码,(例如模板为index.html)
        $str=file_get_contents($filename);
         
        /*$str里面的代吗内容
        <html>
        <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        </head>
        <body>
        <div>{$aa}</div>
        </body>
        </html>
        */
         
        //3.用正则去匹配字符串中出现的{}里面的内容
         
        //4.将内容读取（读取到的是数组里面的key），拿key去数组attr里面取value值
         
            /*$str里面的代码内容
            <html>
            <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            </head>
            <body>
            <div><?php echo $attr[key]?></div>
            </body>
            </html>
            */
         
        //5.将str里面的内容保存在缓存文件里面
        file_put_contents($filename,$str);//$filename是新的文件
         
        //6.将存储的文件加载到当前页面
        include(filename);
    }
     
}
```

## Memcached使用与实践


## redis使用与实践


## hash算法及数据库实现

