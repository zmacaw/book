---
title: 容器入门知识
---

docker 容器遇到的错误处理案例。

# standard_init_linux.<go:178>: exec user process caused "exec format error"

设置entrypoint为启动脚本sh后运行发现报错，最后发现是因为编写的sh脚本不规范造成的，第一行未申明：`#!/bin/bash`

[参考链接stackoverflow](https://stackoverflow.com/questions/42494853/standard-init-linux-go178-exec-user-process-caused-exec-format-error)
