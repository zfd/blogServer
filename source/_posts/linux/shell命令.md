---
title: shell命令
date: 2018-03-01 14:15:20
categories: linux
tags: [linux, shell]
---

```
#!bin/sh
#md5当前目录所有xxx.sql=>xxx.sql.md5

for file in *
do
    if test -f $file
    then
        fName=${file%.*}
        fSuffix=${file##*.}
        if [ "$fSuffix" = "sql" ]
        then
           md5sum $file | cut -d ' ' -f 1 > "$file".md5
        fi  
    fi  
done
```

