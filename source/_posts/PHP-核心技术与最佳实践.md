---
title: PHP核心技术与最佳实践
date: 2018-03-01 10:02:32
categories: 笔记
tags: [php, note]
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

## PHP模板引擎的原理和实践

## Memcached使用与实践

## redis使用与实践

## hash算法及数据库实现


