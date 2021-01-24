---
title: go基础
---

go基础知识。

# Golang教程

[Goalng](http://www.topgoer.com/%E5%85%B6%E4%BB%96/%E5%AE%9E%E6%97%B6%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6%E5%86%85%E5%AE%B9.html)

# 报错处理

1\. go build/mod报错checksum mismatch :

    # go: verifying github.com/docker/docker@v1.13.1: checksum mismatch
    # 解决办法
    go clean -modcache
    cd project && rm go.sum
    go mod tidy
