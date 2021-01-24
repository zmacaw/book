******************
k8s-kubernetes
******************
k8s的安装部署以及的相关知识。

`学习 Kubernetes 基础知识 <https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/>`_ 
--------------------------------------------------------------------------------------------

|create| 创建集群
^^^^^^^^^^^^^^^^^^^^^

.. |create| image:: https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_01.svg?v=1469803628347
    :height: 30px
    :width: 30px
    :scale: 100 %
    :alt: 创建集群
    :align: middle

集群图:

.. image:: https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg

.. tip::
   #若要检查你的 Linux 是否支持虚拟化技术，请运行下面的命令并验证输出结果是否不为空:
   grep -E --color 'vmx|svm' /proc/cpuinfo

1. 安装kubectl:

     a) 手动安装
        
        .. code-block:: bash

			# 使用下面命令下载最新的发行版本：
			curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
			# 要下载特定版本， $(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt) 部分替换为指定版本。

			# 例如，要下载 Linux 上的版本 v1.19.0，输入：
			curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.19.0/bin/linux/amd64/kubectl
			
			# 标记 kubectl 文件为可执行：
			chmod +x ./kubectl

			# 将文件放到 PATH 路径下：
			sudo mv ./kubectl /usr/local/bin/kubectl

			# 测试你所安装的版本是最新的：
			kubectl version --client

     b) 使用包管理器安装
         
        .. code-block:: bash

			sudo apt-get update && sudo apt-get install -y apt-transport-https
			curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
			echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
			sudo apt-get update
			sudo apt-get install -y kubectl

     c) 验证kubctl设置

        kubectl 需要一个  `kubeconfig 配置文件 <https://kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/>`_  使其找到并访问 Kubernetes 集群。 当你使用 kube-up.sh 脚本创建 Kubernetes 集群或者部署 Minikube 集群时，会自动生成 kubeconfig 配置文件。        

        .. code-block:: bash

		# 通过获取集群状态检查 kubectl 是否被正确配置：
		kubectl cluster-info

		# 如果你看到一个 URL 被返回，那么 kubectl 已经被正确配置，能够正常访问你的 Kubernetes 集群。
		# 如果你看到类似以下的信息被返回，那么 kubectl 没有被正确配置，无法正常访问你的 Kubernetes 集群。

		The connection to the server <server-name:port> was refused - did you specify the right host or port?
		# 例如，如果你打算在笔记本电脑（本地）上运行 Kubernetes 集群，则需要首先安装 minikube 等工具，然后重新运行上述命令。

		# 如果 kubectl cluster-info 能够返回 URL 响应，但你无法访问你的集群，可以使用下面的命令检查配置是否正确：
		kubectl cluster-info dump

     d) 启用自动补全

          1. 在 ~/.bashrc 文件中源引自动补齐脚本::
             
             echo 'source <(kubectl completion bash)' >>~/.bashrc

          2. 将自动补齐脚本添加到目录 /etc/bash_completion.d::

          	 kubectl completion bash >/etc/bash_completion.d/kubectl

          3. 如果你为 kubectl 命令设置了别名（alias），你可以扩展 Shell 补齐，使之能够与别名一起使用::
             
             echo 'alias k=kubectl' >>~/.bashrc
			 echo 'complete -F __start_kubectl k' >>~/.bashrc

3. 查看版本::
   
	minikube version

3. 启动::

参考链接:

- `安装Minikube <https://kubernetes.io/zh/docs/tasks/tools/install-minikube/>`_ 

|deploy| 部署应用
^^^^^^^^^^^^^^^^^^^^^

.. |deploy| image:: https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_02.svg?v=1469803628347
    :height: 30px
    :width: 50px
    :scale: 100 %
    :alt: 部署应用
    :align: bottom



|deploy| 部署应用
^^^^^^^^^^^^^^^^^^^^^

.. |deploy| image:: https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_02.svg?v=1469803628347
    :height: 30px
    :width: 50px
    :scale: 100 %
    :alt: 部署应用
    :align: bottom