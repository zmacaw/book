******************
nginx basic
******************
介绍nginx的基本知识和命令

绑定端口
=============
.. code-block:: bash

	#1、端口小于1024，需要以root权限运行
	#2、端口大于1024，Linux默认开启SElinux 导致不能绑定端口
	# 安装管理工具
	yum -y install policycoreutils-python.x86_64
	semanage port -l | grep http_port_t # 查看http允许访问的端口
	semanage port -a -t http_port_t  -p tcp 8090 # 将要启动的端口加入到如上端口列表中