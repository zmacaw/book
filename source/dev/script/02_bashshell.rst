**********************
bourne againe shell
**********************
bash shell 脚本知识和案例。

shell script obfuscation
========================
1. `shc github <https://github.com/neurobin/shc>`_ ::
   
    #如果不加 -r参数，编译后移植到其他系统将无法运行
    shc -e 21/07/2020 -m "Please contact your provider zm" -r -f ./t.sh -o scan

2. gzexe
   
   
dirb dir scanner
=================

.. code-block:: bash
	
	#!/bin/bash 

	while [[ read line ]]; do
	    #statements
	    dirb $line dirspecial
	done < domain.txt

clair scanner
=============

cliarctl批量扫描镜像::

	#!/bin/bash
	if [[ -z $1 || -z $3 || $1 != *.* ]]; then
	        #statements
	        echo "使用方法: ./scanner 仓库地址 镜像列表 clair路径"
	        echo "eg: ./dockerscan hub.docker.com images.txt ./clairctl"
	        exit        
	fi

	n=1
	RED='\033[0;31m'
	NC='\033[0m'
	while IFS= read -r line; do
	        #statements
	        echo -e "\nscan image ${RED}$n${NC}"
	        echo "pull $line"
	        # pull image
	        sudo docker pull "$1/$line" > /dev/null 2>&1 
	        echo "push $line"
	        # push image to clair
	        sudo $3 push --local "$1/$line" >/dev/null 2>&1 
	        echo "scan $line"
	        # analyze image
	        sudo $3 analyze --local "$1/$line" > /dev/null 2>&1 
	        # make report
	        sudo $3 report --local "$1/$line" > /dev/null 2>&1 
	        # delete image from clair
	        sudo $3 delete --local  "$1/$line" > /dev/null 2>&1 
	        # delete image from docker
	        sudo docker rmi "$1/$line" >/dev/null 2>&1 
	        let n+=1
	done < "$2"
