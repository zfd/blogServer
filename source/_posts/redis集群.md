---
title: redis集群
date: 2018-04-17 13:33:16
categories: [服务端]
tags: [redis]
---

> redis cluster：由多个服务于一个数据集合的redis实例组成的整体，redis实例分布在不同服务器。

## 集群特点

+ 所有节点相互连接；
+ 集群消息通过集群总线通信，集群总线端口为客户端端口+10000（固定值）；
+ 节点与节点之间通过二进制协议进行通信；
+ 客户端与节点之间通信和平常一样，通过文本协议进行；
+ 集群节点不会代理查询；
+ 数据按照slot存储分布在多个redis实例上；
+ 集群节点挂掉会自动故障转移；
+ 可以相对平滑扩容、缩容；

## 数据结构

核心结构在cluster.h中定义：

```
// 集群状态，每个节点都保存着一个这样的状态，记录了它们眼中的集群的样子。
// 另外，虽然这个结构主要用于记录集群的属性，但是为了节约资源，
// 有些与节点有关的属性，比如 slots_to_keys 、 failover_auth_count 
// 也被放到了这个结构里面。
typedef struct clusterState {
    ...
    // 指向当前节点的指针
    clusterNode *myself;

    // 集群当前的状态：是在线还是下线 REDIS_CLUSTER_OK, REDIS_CLUSTER_FAIL, ...
    int state;

    // 集群节点名单（包括 myself 节点）
    // 字典的键为节点的名字，字典的值为 clusterNode 结构
    dict *nodes;

    // 记录要从当前节点迁移到目标节点的槽，以及迁移的目标节点
    // migrating_slots_to[i] = NULL 表示槽 i 未被迁移
    // migrating_slots_to[i] = clusterNode_A 表示槽 i 要从本节点迁移至节点 A
    clusterNode *migrating_slots_to[REDIS_CLUSTER_SLOTS];

    // 记录要从源节点迁移到本节点的槽，以及进行迁移的源节点
    // importing_slots_from[i] = NULL 表示槽 i 未进行导入
    // importing_slots_from[i] = clusterNode_A 表示正从节点 A 中导入槽 i
    clusterNode *importing_slots_from[REDIS_CLUSTER_SLOTS];

    // 负责处理各个槽的节点
    // 例如 slots[i] = clusterNode_A 表示槽 i 由节点 A 处理
    clusterNode *slots[REDIS_CLUSTER_SLOTS];

    // 跳跃表，表中以槽作为分值，键作为成员，对槽进行有序排序
    // 当需要对某些槽进行区间（range）操作时，这个跳跃表可以提供方便
    // 具体操作定义在 db.c 里面
    zskiplist *slots_to_keys;
    ...
} clusterState;

// 节点状态
struct clusterNode {
    ...
    // 节点标识
    // 使用各种不同的标识值记录节点的角色（比如主节点或者从节点），
    // 以及节点目前所处的状态（比如在线或者下线）。
    int flags;

    // 由这个节点负责处理的槽
    // 一共有 REDIS_CLUSTER_SLOTS / 8 个字节长
    // 每个字节的每个位记录了一个槽的保存状态
    // 位的值为 1 表示槽正由本节点处理，值为 0 则表示槽并非本节点处理
    // 比如 slots[0] 的第一个位保存了槽 0 的保存情况
    // slots[0] 的第二个位保存了槽 1 的保存情况，以此类推
    unsigned char slots[REDIS_CLUSTER_SLOTS/8];

    // 指针数组，指向各个从节点
    struct clusterNode **slaves;

    // 如果这是一个从节点，那么指向主节点
    struct clusterNode *slaveof;
    ...
};

// clusterLink 包含了与其他节点进行通讯所需的全部信息
typedef struct clusterLink {
    ...
    // TCP 套接字描述符
    int fd;

    // 与这个连接相关联的节点，如果没有的话就为 NULL
    struct clusterNode *node;
    ...
} clusterLink;
```

## 集群通信

### 通信端口

集群中每个redis实例，都会使用两个tcp端口：

+ 一个给客户端（redis-cli或应用程序等）使用；
+ 另一个用于集群中实例相互通信的内部总线端口，此端口比第一个大10000。

### 通信协议

> Redis集群采用P2P的Gossip（流言）协议，Gossip协议工作原理就是节点彼此不断通信交换信息，一段时间后所有的节点都会知道集群完整的信息，这种方式类似流言传播。

+ 集群中每个节点通过一定规则挑选要通信的节点，每个节点可能知道全部节点，也可能仅知道部分节点，只要这些节点彼此可以正常通信，最终它们会达到一致的状态。
+ 当节点出故障、新节点加入、主从角色变化、槽信息变更等事件发生时，通过不断的ping/pong消息通信，经过一段时间后所有的节点都会知道整个集群全部节点的最新状态，从而达到集群状态同步的目的。

### 组建集群（cluster meet）

> 组建集群，把各个独立的节点链接起来，构成一个包含多个节点的集群。

命令：cluster meet <ip> <port>

1. 客户端向节点A发送cluster meet <nodeB ip> <nodeB port>命令；
1. 节点A为节点B创建一个clusterNode结构，并添加到clusterState.nodes字典里面，再向节点B发送meet消息；
1. 节点B收到meet消息，会为节点A添加一个clusterNode结构，并添加到clusterState.nodes字典里面，再向节点A返回一条pong消息；
1. 节点A收到pong消息，再向节点B发送ping消息；
1. 节点B收到ping消息，握手完成；
1. 之后，节点A会将节点B的信息通过Gossip协议传播给集群中的其他节点，让其他节点也跟节点B握手；
1. 最终，节点B会被集群中所有节点认识。

### 消息处理（clusterProcessPacket）

+ 更新接收消息计数器
+ 查找发送者节点并且不是handshake节点
+ 更新自己的epoch和slave的offset信息
+ 处理MEET消息，使加入集群
+ 从goosip中发现未知节点，发起handshake
+ 对PING，MEET回复PONG
+ 根据收到的心跳信息更新自己clusterState中的master-slave，slots信息
+ 对FAILOVER_AUTH_REQUEST消息，检查并投票
+ 处理FAIL，FAILOVER_AUTH_ACK，UPDATE信息

### 定时任务（clusterCron）

+ 对handshake节点建立Link，发送Ping或Meet
+ 向随机几点发送Ping
+ 如果是从查看是否需要做Failover
+ 统计并决定是否进行slave的迁移，来平衡不同master的slave数
+ 判断所有pfail报告数是否过半数

### 心跳数据

发送header信息：

+ 所负责的slots的信息
+ 主从信息
+ ip port信息
+ 状态信息

发送其他节点gossip信息：

+ ping_sent,pong_received
+ ip、port信息
+ 状态信息

## 分区原理

### 槽（slot）

+ 是一个虚拟的槽，总长度16384，为固定值，编号0~16383（用户可以配置）；
+ 每个master节点负责一部分槽；
+ 用户get/set时，先查找对应槽位，crc16(key)%16384，再查找对应节点，从而实现负载均衡。

注：crc，循环冗余校验（Cyclic Redundancy Check）

### 位序列结构

+ 每个master节点维护一个16384/8=2048个unsigned int长度的位序列，用于检查某个槽是否拥有；
+ 还维护一个槽到集群节点的映射，长度为16384的数组，数组下标为槽编号，数组的值为集群节点，用于快速查找槽所在的节点；

### 键哈希标签原理

将一批数据放入同一个槽中，只需要按规则生成key，redis计算时只处理花括号内字符串：

```
//如：设置2个key为，abc{userId}def，ghi{userId}jkl

unsigned int keyHashSlot(char *key, int keylen) {
    int s, e; 
    for (s = 0; s < keylen; s++)
        if (key[s] == '{') break;
 
    if (s == keylen) return crc16(key,keylen) & 0x3FFF;
 
    for (e = s+1; e < keylen; e++)
        if (key[e] == '}') break;
 
    if (e == keylen || e == s+1) return crc16(key,keylen) & 0x3FFF;
    return crc16(key+s+1,e-s-1) & 0x3FFF;
}
```

### 请求重定向

### moved错误

1. 请求的key对应的槽不在该节点上，节点将查看自身所保存的哈希槽到节点的映射记录，返回一个moved错误；
1. 客户端需要再次向新节点重试。

### ask错误

1. 请求的key对应的槽目前属于migrating状态，并且当前节点找不到这个key了，节点返回ask错误。
1. ask会把对应槽的importing节点返回，让客户端重试；
1. 客户端重试，先发送asking命令，节点将为客户端设置一个一次性的标志（flag），使得客户端可以执行一次针对importing状态的槽的命令请求，然后再发送真正的命令请求；
1. 不必更新客户端所记录的槽至节点的映射。

## 数据迁移

## 通信故障

集群中每个节点都会定期向集群中的其他节点发送ping消息，以此交换各个节点状态信息，检查各个节点状态。

节点状态：在线、怀疑下线、已下线。

### 主节点状态fail

集群里面，超过半数以上的主节点都将某主节点A报告为怀疑下线，那么A将被标记为已下线，并且标记他的节点处，会向集群广播他的fail消息。

#### 多个从节点选主

主节点，选举方式进行

#### 故障转移

1. 从下线主节点的所有从节点中选中一个从节点
1. 被选中的从节点执行SLAVEOF NO NOE命令，成为新的主节点
1. 新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己
1. 新的主节点对集群进行广播PONG消息，告知其他节点已经成为新的主节点
1. 新的主节点开始接收和处理槽相关的请求

### 集群状态fail

+ 集群中任意master节点挂掉，并且该master没有slave，集群计入fail状态；
+ 集群超过半数以上master挂掉，无论是否有slave，集群进入fail状态。 
