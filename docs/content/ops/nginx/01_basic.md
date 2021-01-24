---
title: nginx basic
---

介绍nginx的基本知识和命令

# 绑定端口

``` bash
#1、端口小于1024，需要以root权限运行
#2、端口大于1024，Linux默认开启SElinux 导致不能绑定端口
# 安装管理工具
yum -y install policycoreutils-python.x86_64
semanage port -l | grep http_port_t # 查看http允许访问的端口
semanage port -a -t http_port_t  -p tcp 8090 # 将要启动的端口加入到如上端口列表中
```
# 配置443端口和自签名证书

**自签名证书生成请参考容器安全扫描部分**，使用自己的证书路径替换配置中server.crt和server.key文件路径。

``` conf
server {

        listen       443 ssl;
        server_name  localhost;
        ssl_certificate     /root/Lee/keys/server.crt;#配置证书位置
        ssl_certificate_key  /root/Lee/keys/server.key;#配置秘钥位置
        #ssl_client_certificate ca.crt;#双向认证
        #ssl_verify_client on; #双向认证
 
        ssl_session_timeout  5m;
        ssl_protocols  SSLv2 SSLv3 TLSv1.2;
        ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        ssl_prefer_server_ciphers   on;
        ```