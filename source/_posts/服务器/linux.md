---
title: linux
date: 2018-04-18 22:34:46
categories: [服务器]
tags: [linux]
---

## 权限

+ 权限是操作系统用来限制资源访问的机制，权限一般分为读、写、执行。
+ 系统中每个文件都拥有特定的权限、所属用户及所属组，通过这样的机制来限制哪些用户、哪些组可以对特定的文件进行什么样的操作。
+ 每个进程都是以某个用户的身份运行，所以进程的权限与该用户的权限一样，用户的权限越大，该进程所拥有的权限也就越大。

|权限|对文件的影响|对目录的影响|
|---|---|---|
|r，读取|可读取文件内容|可列出目录内容|
|w，写入|可修改文件内容|可在目录创建、删除文件|
|x，执行|可作为命令执行|可访问目录内容|

注：目录必须拥有'x'权限才可查看其内容。

### UGO模型

+ U，代表user，所属用户权限；
+ G，代表group，所属用户组权限；
+ O，代表other，其他用户的权限；

### 权限设置

+ 每个文件/目录权限基于UGO设置；
+ 权限三个一组（rwx），对应UGO分别设置；（即共有3组，9个权限）；

例子：

```
$ ls -l
total 280
drwxr-xr-x    1 zfd  staff      18 Feb 18 17:34 README.md
drwxr-xr-x    1 zfd  staff    1850 Feb 26 20:50 _config.yml
drwxr-xr-x    1 zfd  staff     174 Apr 18 22:34 db.json
drwxr-xr-x  321 zfd  staff   10914 Feb 22 22:01 node_modules
drwxr-xr-x    1 zfd  staff  124449 Feb 22 22:01 package-lock.json
drwxr-xr-x    1 zfd  staff     743 Feb 18 18:06 package.json
drwxr-xr-x   13 zfd  staff     442 Mar 28 00:43 public
drwxr-xr-x    5 zfd  staff     170 Feb 18 17:34 scaffolds
drwxr-xr-x    5 zfd  staff     170 Feb 18 17:34 source
drwxr-xr-x    3 zfd  staff     102 Feb 18 17:34 themes
```

说明：

| -rw-r--r-- | 1 | zfd | staff | 18 | Feb 18 17:34 | README.md |
|---|---|---|---|---|---|---|
|UGO模型的权限|链接数量|用户U|用户组G|大小|时间|文件/目录名|

drwxr-xr-x：

+ 第1位"d"是文件类型描述符，"d"是目录，"-"是文件；
+ 2-4位"rwx"是U模型权限，这里表示可读、可写、可执行；
+ 4-7位"r-x"是G模型权限，这里表示可读、不可写，可执行；
+ 8-10位"r-x"是O模型权限，这里表示可读、不可写，可执行。

### 修改文件权限命令

mode参数格式：

+ u、g、o，分表代表用户、组、其他；
+ a，代表ugo；
+ +、-，代表加入或删除对应权限；
+ r、w、x，代表三种权限；
+ -R，代表递归地修改；

例子：

```
//chmod mode fileName

chmod u +rw test.md    给文件的所属用户添加rw权限
chmod g -x test.md     给文件的所属组移除x权限
chmod go +r test.md    给文件的所属组和其他用户添加r权限
chmod a -x test.md     给文件的所属UGO三个模型均移除x权限
```

也支持3位8进制数值的方式修改：

```
//八进制
r = 4 (2 ^ 2)
w = 2 (2 ^ 1)
x = 1 (2 ^ 0)

rw- = 4 + 2 + 0 = 6    二进制的110
rwx = 4 + 2 + 1 = 7    二进制的111
r-x = 4 + 0 + 1 = 5    二进制的101

chmod 660 test.md  设置 UGO 权限为 rw-rw----
chmod 777 test.md  设置 UGO 权限为 rwxrwxrwx
```
