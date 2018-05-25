---

title: git使用
date: 2018-05-25 15:00:00
categories: 日常
tags: [git]

---

## git bash 自动认证

### 生成ssh key

```
cd ~
ssh-keygen -t rsa -C "zf.dong@qq.com" //对应git邮箱账号
cat ~/.ssh/id_rsa.pub //复制此处
```

### 添加到对应网站

#### github

Setting -> SSH and GPG keys -> New SSH key -> Add key

#### gitlab

User Setting -> SSH Keys -> Add key

## git tortoise 自动认证

### 生成ssh key && putty key

+ tortoiseGit安装目录下，bin/puttygen.exe -> generate；
+ 复制key，放到对应网站add key；
+ save private key -> xxx.ppk；
+ 进入git项目目录，tortoiseGit -> Setting -> Git -> 远端 -> 添加 putty 秘钥（xxx.ppk）；
