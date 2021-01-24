---
title: linux基础
---

这一部分主要介绍elbk的相关知识和案例。

# git
1. git报错：

# Linux输入法设置

**ibus中文输入法也很好用**

最近安装了kali2020.02虚拟机（官方VirtualBox镜像），发现默认是XFCE桌面（轻量级、简单），但是还是习惯关Gnome，于是使用以下方法切换到了Gnome桌面。

# 安装Gnome桌面

``` bash
# 安装Gnome组件
sudo tasksel

# 设置Gnome为默认session
sudo update-alternatives --config x-session-manager
```

安装Gnome桌面发现我的goolge pinyin用不了了，fcitx配置里边空空如也，经过一番搜索，猜测估计是Gnome安装了ibus，默认使用了ibus，
fcitx 是 Free Chinese Input Toy for X 的简称。

源代码托管在 [Gitlab](<https:#gitlab.com/fcitx)

在 Debian/Ubuntu 下可以使用

``` bash
apt install fcitx
apt install fcitx-pinyin    # 汉语拼音方案
apt install fcitx-googlepinyin
apt install fcitx-hangul    # 韩语
apt install fcitx-rime      # 小狼毫
```

# curl SSL报错解决


After regular update and upgrade of Debian OS curl stop working on specific sites: curl: (35) error:141A318A:SSL routines:tls_process_ske_dhe:dh key too small

``` bash
# 解决办法：
# 编辑 /etc/ssl/openssl.cnf注释掉以下行：

CipherString = DEFAULT@SECLEVEL=2
```

# firewall

1.  禁止指定IP的连接:

        # drop会导致客户端连接超时(建议在公网使用)  reject直接拒绝客户端连接(建议在内网使用，便于排查)
        firewall-cmd --zone=public --add-rich-rule="rule family='ipv4' source address='10.0.0.1/24' port port=80 protocol=tcp drop" --permanent

2.  开放端口:

        # --permanent永久生效，需要重新reload firewalld才会生效，不加此参数立即生效(可以通过firewall-cmd --list-all查看到)
        firewall-cmd --zone=public --add-port=5601/tcp --permanent

3.  端口转发:

        #本地转发 
        firewall-cmd --add-forward-port=port=port-number:proto=tcp|udp|sctp|dccp:toport=port-number
        #转发到其他IP
        firewall-cmd --add-forward-port=port=port-number:proto=tcp|udp:toport=port-number:toaddr=IP/mask
        # 参考[RedHat](https:#access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-port_forwarding) 
        #启用masquerade
        #iptables中DNAT、SNAT和MASQUERADE的理解https:#www.centos.bz/2017/09/iptables%E4%B8%ADdnat%E3%80%81snat%E5%92%8Cmasquerade%E7%9A%84%E7%90%86%E8%A7%A3
        firewall-cmd --add-forward-port=port=80:proto=tcp:toport=888


4.  永久生效:

        firewall-cmd --runtime-to-permanent

# iptables

1.  端口转发:

        sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222

# 文件操作

1. 批量递归操作文件

        find . -iname "*.rst" | while read i; do pandoc -s -o "${i%.*}.md" "$i"; done

# ls按文件大小排序