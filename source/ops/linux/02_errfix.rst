******************
linux故障处理
******************
这一部分主要介绍Linux故障处理的相关知识和案例。


/lib64/libc.so.6: version `GLIBC_2.14' not found
==================================================

1. 安装 `下载地址 <http://www.gnu.org/software/libc/>`_ 
::
    wget http://ftp.gnu.org/gnu/glibc/glibc-2.14.tar.gz
    tar -zxvf glibc-2.14.tar.gz
    cd glibc-2.14
    mkdir build
    cd build
    ../configure --prefix=/opt/glibc-2.14
    make -j4
    sudo make install
    export LD_LIBRARY_PATH=/opt/glibc-2.14/lib (for current login session) OR add LD_LIBRARY_PATH=/opt/glibc-2.14/lib in the /etc/environment and perform source /etc/environment(to add env variable permanently)


.. warning:: 这里环境变量要如上一样临时修改，决不能写在/etc/profile文件里，并source使之生效！否则会导致某些shell命令执行不了。

2. glibc软链::
   
	rm -rf /lib64/libc.so.6           // 先删除先前的libc.so.6软链
	ln -s /opt/glibc-2.14/lib/libc-2.14.so /lib64/libc.so.6


.. warning:: 删除libc.so.6之后可能导致系统命令不可用的情况,这时候要特别注意：千万不要关闭当前的终端窗口！！！因为此时机器可能无法登陆了,只能在当前终端窗口下进行紧急修复 可使用如下方法解决::

	LD_PRELOAD=/opt/glibc-2.14/lib/libc-2.14.so ln -s /opt/glibc-2.14/lib/libc-2.14.so /lib64/libc.so.6


.. warning:: 如果上述更新失败可使用如下命令还原::
	
	LD_PRELOAD=/lib64/libc-2.12.so ln -s /lib64/libc-2.12.so /lib64/libc.so.6 
	// libc-2.12.so 是系统升级前的版本