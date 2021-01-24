---
title: linux脚本工具
---

# 判断文件夹大小并清除文件

```shell
#!/bin/bash

F1="/data/"
F2="/data/app/*"
F3="/data/files/*"
LIMIT="180000000"

D1=$(du -s $F1 | awk '{print $1}')

if [ "$D1" -gt $LIMIT ]
  then
    rm -rf $F2
    rm -rf $F3
fi

```