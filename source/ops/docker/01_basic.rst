******************
容器入门知识
******************
docker的安装部署以及docker registry的相关知识。

`Docker-registry-API <https://docs.docker.com/registry/spec/api/>`_ 
---------------------------------------------------------------------------

1. 查看镜像清单 ::

	http://docker.io/v2/_catalog?n=

3. 查看镜像标签 ::

	http://docker.io/v2/<imageName>/tags/list?n= //n表示要列出的数量

4. 拉取镜像配置 ::
	
	http://docker.io/v2/<imageName>/manifests/latest<Reference>  //reference可以是标签或者digest

5. 按层拉取镜像 :: 

	http://docker.io/v2/<image>/blobs/<digest> //digest是镜像每个fsLayer层的唯一标识。存在于上一步下载的配置的fsLayers里面。

docker网络
---------------------

1. docker使用主机网络::
   
   --net=host

docker启动任务(1号进程)
--------------------------

1. 启动多个任务可以将需要启动的任务写到一个sh脚本中（脚本需要规范，不然会报错），再将脚本设置为启动的entrypoint

容器/镜像导入导出
--------------------------

.. warning:: 容器的导入导出不会导出数据卷等信息，会造成导入后无法运行的情况，很多文件会找不到，并且导出后的容器运行时需要加上运行命令【通过docker ps -a --no-trunc可以查看到 COMMAND列】

1. 容器导入导出::
   
   # 导出语法
	docker export [OPTIONS] CONTAINER
	# 例子
	docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2

	# 导入语法
	docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
	# 例子
	docker import  my_ubuntu_v3.tar runoob/ubuntu:v4


2. 镜像导入导出
   
   # 导出语法
	docker save [OPTIONS] IMAGE [IMAGE...]
	# 例子，如果需要跨操作系统，请使用 -o 方式
	docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
	docker save runoob/ubuntu:v3 > my_ubuntu_v3.tar

	# 导入语法
	docker load [OPTIONS]
	# 例子，如果需要跨操作系统，请使用 -i 方式
	docker load -i ubuntu.tar
	docker load < ubuntu.tar

.. tip:: docker save 保存的是镜像，docker export 保存的是容器
		 docker load 用来载入镜像包，docker import 用来载入容器包，但两者都会恢复为镜像
		 docker load 不能对载入的镜像重命名，而 docker import 可以为镜像指定新名称



docker数据管理
-------------------------
按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者 绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。
数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

1. docker数据卷（volume）
 
.. code-block:: bash
   
   //查看所有数据卷
   docker volume ls

   //创建数据卷
   docker volume create my-vol

   //查看指定数据卷信息（挂载点等）
   docker volume inspect my-vol

   //下面创建一个名为 web 的容器，并加载一个 数据卷 到容器的 /usr/share/nginx/html 目录。
   docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine

    //查看容器挂载的数据卷信息【在Mounts节点下】
    docker inspect web

    //删除数据卷
    docker volume rm my-vol


    //删除容器时同时删除数据卷
    docker rm -v

    //无主的数据卷可能会占据很多空间，要清理请使用以下命令
    docker volume prune

2. 挂载主机目录
   
.. code-block:: bash

   //加载主机的 /src/webapp 目录到容器的 /usr/share/nginx/html目录【本地目录必须使用绝对路径】
   docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine

    //挂载为只读（,readonly）
    docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html:ro \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine

    //查看容器挂载信息【在Mounts节点下】
    docker inspect web

    //挂载主机文件到容器中
    docker run --rm -it \
	   # -v $HOME/.bash_history:/root/.bash_history \
	   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
	   ubuntu:18.04 \
	   bash
	//这样就可以记录在容器中输入的命令了
	root@2affd44b4667:/# history
	1  ls
	2  diskutil list