******************
蜜罐
******************
企业级蜜罐的部署和配置。

[T-pot](https://github.com/dtag-dev-sec/tpotce)安装部署
=======================================================

安装部署后如果tpot服务启动失败，可尝试手动安装。
先切换到root用户：sudo su -
1.安装apt-fast
::

	git clone https://github.com/ilikenwf/apt-fast.git
	cd apt-fast
	sudo cp apt-fast /usr/local/sbin/
	sudo chmod +x /usr/local/sbin/apt-fast
	sudo cp apt-fast.conf /etc
	sudo apt-get install aria2 -y

2.安装docker
::

	cd /opt/tpot/
	sudo ./install.sh --type=user

[HFish](https://hfish.io/)安装部署
====================================

1、修改ssh端口为其他端口,修改ssh端口报错解决办法::

	semanage port -a -t ssh_port_t -p tcp 2222

2、开放蜜罐端口::

	semanage port -a -t ssh_port_t -p tcp 2222
	firewall-cmd --zone=public --add-port=2222/tcp --permanent 
	firewall-cmd --zone=public --add-port=8080/tcp --add-port=9000/tcp  --add-port=21/tcp  --add-port=22/tcp  --add-port=23/tcp  --add-port=445/tcp  --add-port=3389/tcp  --add-port=135/tcp  --add-port=3306/tcp  --add-port=6379/tcp --add-port=8080/tcp --add-port=11211/tcp --add-port=69/tcp --add-port=5900/tcp --add-port=8081/tcp --add-port=9200/tcp

[MHN](https://github.com/pwnlandia/mhn)安装部署
===============================================

禁止数据上报
MHN Server会默认将分析数据上报给Anomali，如果需要禁用此配置，运行如下命令::

	cd mhn/scripts/
	sudo ./disable_collector.sh
