******************
kibana
******************
介绍kibana


kibana设置登录
================
Kibana从5.5开始不提供认证功能，想用官方的认证X-Pack，收费滴

所以自己动手，使用Nginx的代理功能
安装Nginx
安装Apache密码生产工具

.. code-block:: bash

    
	yum install httpd-tools
	yum install nginx

	生成密码文件

	mkdir -p /etc/nginx/passwd
	htpasswd -c -b /etc/nginx/passwd/kibana.passwd user ******

	#配置Nginx

	vi /etc/nginx/nginx.conf

	server {
	  listen 192.168.10.10:5601;

	  auth_basic "Kibana Auth";
	  auth_basic_user_file /etc/nginx/passwd/kibana.passwd;

	  location / {
	    proxy_pass http://127.0.0.1:5601;
	    proxy_redirect off;
	  }
	}

	#修改Kibana配置文件

	vim /etc/kibana/kibana.yml

	# The host to bind the server to.
	server.host: "localhost"

	#重启Kibana服务，配置文件生效

	ps -ef | grep node
	kill -9 xxxx
	nohup ./bin/kibana >/dev/null &

	# 重载Nginx配置

	nginx -s reload

	# 登录Kibana


kibana-Request Timeout after 30000ms故障解决
---------------------------------------------
1. 如果机器的内存还是毕竟充足的话，那么就给elasticsearch多一点内存,配置文件/etc/elasticsearch/jvm.options
2. 如果机器的内存不是那么的充足的话，我们可以改改后端弹性搜索的阈值。修改配置文件/etc/kibana/kibana.yml的第66行，将#去掉，然后将30000毫秒（也就是30s）更改成40000(40秒)，这个根据实际情况进行修改。
