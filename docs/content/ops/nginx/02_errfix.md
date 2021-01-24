---
title: nginx basic
---

介绍nginx的基本知识和命令

# failed (13: Permission denied) while connecting to upstream

nginx设置proxy_pass无法访问代理地址。

**原因：**
是因为SeLinux的限制

**解决办法**
1. 关闭SeLinux
2. semanage port -a -t http_port_t  -p tcp 8090 # 将要访问的端口加入到端口列表中