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

## 缓存详解

> `缓存`：凡是位于速度相差较大的两种介质之间，用于协调两者数据传输速度差异的结构。

### 作用

+ 性能：解决高并发，大数据场景下，热点数据访问的性能问题。提供高性能的数据快速访问。(主要作用)
+ 稳定性：一些重复请求每次都要处理，会加大服务器的资源消耗，影响系统稳定性。
+ 可用性：提供数据的服务挂了，缓存可以在一段时间内正常提供用户支持，提高系统可用性。

### 原理

减少计算量，缩短请求流程（网络i/o、硬盘i/o）：

+ 将数据写入/读取速度更快的存储（设备）；
+ 将数据缓存到离应用最近的位置；
+ 将数据缓存到离用户最近的位置。

### 应用

理论上，web的每一层都可以被缓存。

+ CPU缓存；
+ 内存：Memcached这样的Key Value内存缓存；
+ 数据库：Query cache，Table cache，Thread cache；
+ 应用程序代码级别：Smarty文件缓存；
+ 浏览器：浏览器缓存；

注：理论上各层都可以被缓存。

`缓存的存放位置`：CPU、内存、硬盘。

### 缓存的三个要素

> 命中率，更新策略，最大数据量。

#### 命中率

命中率 = 请求缓存次数 / 缓存返回正确结果，比例越高越好，比例过低可能会造成反效果。

### 更新策略

超出缓存最大数据量时，清理缓存的常用策略：

+ FIFO（first in first out），先进先出；（如：mysql的query cache）
+ LFU（less frequently used），最少使用；
+ LRU（least recently used），最近最少使用；

### 最大数据量

缓存元素的最大个数 或 所能使用的最大存储空间。

超出最大数据量时，一般如下处理：

+ 停止缓存服务，所有缓存数据清空；
+ 拒绝写入，拒绝更新；
+ 根据缓存策略，清理旧数据（还可以备份旧数据）；

## Memcached使用与实践

> 本质上是一个key/value的内存数据库，但是不支持数据的持久化，服务器关闭之后数据全部丢失。

### 为什么要用memcached？

+ 对数据库的高并发读写；（关系型数据库承受不了高并发的读写，如：每秒上万次）
+ 对海量数据的处理；（数据量上亿左右，表的读写效率低）

## redis使用与实践

> 本质上是一个key/value的内存数据库。

### 数据结构

| 结构类型 | 结构存储的值 | 结构的读写能力 |
|---|---|---|
| string | 字符串、整形、浮点数 | 对字符串整个或部分操作，对整形、浮点数自增或自减 |
| list | 链表，每个节点都包含一个字符串 | 两端push、pop；根据偏移量修剪；读取单个或多个元素；根据值查找或移除元素 |
| set | 无序集合，包含string，并且每个字符串都是唯一 | CRUD元素；与集合计算交集、并集、差集；从集合随机获取元素 |
| hash | 散列表，包含键值对，key/value 都是字符串类型 | CRUD键值对；获取所有键值对 |
| zset | 有序集合，字符串成语与浮点数分值之间的有序映射，元素排序顺序由分值大小决定 | CRUD元素；根据分值范围或成员获取元素 |

#### 简单动态字符串（SDS）

``` C
/* 
 * 保存字符串对象的结构 
 */  
struct sdshdr {      
    // buf 中已占用空间的长度  
    int len;  
    // buf 中剩余可用空间的长度  
    int free;  
    // 数据空间  
    char buf[];  
};  
```

优点：

+ 常数复杂度获取字符串的长度：len属性保存了字符串的长度；（'\0'空字符不计入len）
+ 杜绝缓冲区溢出：当对SDS进行修改时，先检查SDS的空间是否满足修改所需要的空间要求，如果不满足，则先扩展空间，然后执行修改操作；
+ 减少修改字符串带来的内存重分配次数：
    + 空间预分配：当SDS的长度小于1MB时，分配（2 * len + 1B）的空间；当SDS的长度大于等于1MB时，分配（len + 1MB + 1B）的空间；
    + 惰性空间释放：当缩短字符串时，并不会立即使用内存重分配来回收多出来的字符，而是记录在free属性中；

#### 链表

``` C
/* 
 * 双端链表节点 
 */  
typedef struct listNode {  
    // 前置节点  
    struct listNode *prev;  
    // 后置节点  
    struct listNode *next;  
    // 节点的值  
    void *value;  
} listNode;  

/* 
 * 双端链表结构 
 */  
typedef struct list {  
    // 表头节点  
    listNode *head;  
    // 表尾节点  
    listNode *tail;  
    // 节点值复制函数  
    void *(*dup)(void *ptr);  
    // 节点值释放函数  
    void (*free)(void *ptr);  
    // 节点值对比函数  
    int (*match)(void *ptr, void *key);  
    // 链表所包含的节点数量  
    unsigned long len;  
} list; 
```

#### 字典

```
/* 
 * 字典 
 */  
typedef struct dict {  
    // 类型特定函数，Redis为不同用途的字典设置不同的类型特定函数  
    dictType *type;  
    // 私有数据，传递给特定类型函数的可选参数  
    void *privdata;  
    // 哈希表，一般情况下字典使用ht[0]，ht[1]只会在对ht[0]进行rehash时使用  
    dictht ht[2];  
    // rehash 索引  
    // 当 rehash 不在进行时，值为 -1  
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */  
    // 目前正在运行的安全迭代器的数量  
    int iterators; /* number of iterators currently running */  
} dict;  

/* 
 * 哈希表 
 * 每个字典都使用两个哈希表，从而实现渐进式 rehash 。 
 */  
typedef struct dictht {      
    // 哈希表数组，存放具体的键值对  
    dictEntry **table;  
    // 哈希表大小  
    unsigned long size;      
    // 哈希表大小掩码，用于计算索引值  
    // 总是等于 size - 1  
    unsigned long sizemask;  
    // 该哈希表已有节点的数量  
    unsigned long used;  
} dictht;  

/* 
 * 哈希表节点 
 */  
typedef struct dictEntry {      
    // 键  
    void *key;  
    // 值  
    union {  
        void *val;  
        uint64_t u64;  
        int64_t s64;  
    } v;  
    // 指向下个哈希表节点，形成链表  
    struct dictEntry *next;  
} dictEntry;  
```

#### 跳跃表（skiplist）

``` C
/* 
 * 跳跃表 
 */  
typedef struct zskiplist {  
    // 表头节点和表尾节点  
    struct zskiplistNode *header, *tail;  
    // 表中节点的数量  
    unsigned long length;  
    // 表中层数最大的节点的层数，表头结点的层数不计算在内  
    int level;  
} zskiplist;  

/* 
 * 跳跃表节点 
 */  
typedef struct zskiplistNode {  
    // 成员对象  
    robj *obj;  
    // 分值，跳跃表中节点按各自所保存的分值从小到大排列  
    double score;  
    // 后退指针，指向当前节点的前一个节点  
    struct zskiplistNode *backward;  
    // 层，每次创建一个新节点，程序按幂次定律随机生成一个1~32的值作为level数组的大小（层高度）  
    struct zskiplistLevel {  
        // 前进指针  
        struct zskiplistNode *forward;  
        // 跨度，前进指针所指向的节点和当前节点的距离  
        unsigned int span;  
    } level[];  
} zskiplistNode;  
```

#### 整数集合（intset）

```
typedef struct intset {      
    // 编码方式  
    uint32_t encoding;  
    // 集合包含的元素数量  
    uint32_t length;  
    // 保存元素的数组，各项在数组中按值大小有序地排序并且不包含重得项  
    int8_t contents[];  
} intset;  
```

####  压缩列表（ziplist）


### redis优势

+ Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
+ Redis支持数据的备份，即master-slave模式的数据备份。
+ Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

### 持久化

+ 内存快照（snapshotting）

每个一段时间进行一次内存快照，把内存中的数据写入二进制文件中（*.rdb）；save，在主进程操作，阻塞主进程，不能快速响应请求；bgsave，fork一个子进程操作，主进程继续处理请求，完成后通知主进程；

缺点：每次都是全部内存数据写入，数据量大会操作频繁，影响性能。

+ 日志追加（append only file）

把增加、修改数据的命令通过write函数追加到文件尾部（*.aof）；redis重启时读取所有命令并且执行，从而把数据写入内存；

缺点：日志文件膨胀比较快，如nums自增100次，恢复只需要一次set nums 100，其他命令多余。

优化：bgrewriteaof命令，类似于内存快照方式，把命令保存到临时文件，最后再替换原来的日志文件。

### 主从复制

| master服务器 | slave服务器 |  
|---|---|
| | 1.连接（或重新连接）主服务器，发送同步（SYNC）命令 |
| 2.开始执行bgsave，生成快照文件（.rdb），然后向slave服务器发送，并使用缓冲区记录后续执行的写命令 | |
| | 3.丢弃所有旧数据，接收并载入master服务器发来的快照文件，解释并处理完毕后，正常接收命令 |
| 4.继续发送缓冲区记录的后续写命令；发送完毕后，每执行一个新的写命令就再发送一次 | |
| | 5.接收并执行每一个写命令 |

## hash算法及数据库实现

> 把任意长度的输入，通过hash算法变成固定长度的输出，输出的值就是hash的值。

