---
title: 容器漏洞扫描
---

容器漏洞扫描器clair/anchore的部署和配置。

# cicd-jenkins-anchore-gitlab安装部署

*based on kali-2020.02*

## docker

Debian系统docker安装

``` {.sh}
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## docker-compose

``` {.bash}
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## anchore

*注意，okcer-compose命令必须要在dockers-compose.yaml同级目录下执行*

``` {.bash}
# 下载yaml文件
curl https://docs.anchore.com/current/docs/engine/quickstart/docker-compose.yaml > docker-compose.yaml

#　启动anchore
docker-compose up -d

# 查看启动状态
docker-compose ps

# 查看安装状态（engine-api要根据dockers-compose.yaml中的配置替换，如我所安装的版本中为api，此时engine-api需要替换成api）
docker-compose exec engine-api anchore-cli system status

# 数据库同步状态
docker-compose exec engine-api anchore-cli system feeds list

# 等待漏洞库（feeds）更新完毕
docker-compose exec engine-api anchore-cli system wait

# 开始使用anchore
docker-compose exec engine-api anchore-cli image add docker.io/library/debian:7

docker-compose exec engine-api anchore-cli image wait docker.io/library/debian:7

docker-compose exec engine-api anchore-cli image content docker.io/library/debian:7 os

docker-compose exec engine-api anchore-cli image vuln docker.io/library/debian:7 all

docker-compose exec engine-api anchore-cli evaluate check docker.io/library/debian:7
```

## 安装anchore-cli本地使用cli和anchore engine交互

> sudo apt-get install python3-pip pip3 install anchore-cli
>
> \#
> 如果提示WARNING，anchore-cli未安装在PATH路径下，执行以下命令，创建软链接（必须使用全路径，不能使用相对路径，否则会报错：Too
> many levels of symbolic links） \#
> 根据安装信息，如安装在/home/kali/.local/bin/anchore-cli sudo ln -s
> /home/kali/.local/bin/anchore-cli /usr/bin/anchore-cli
>
> \# 设置环境变量 export ANCHORE_CLI_URL=http://127.0.0.1:8228/v1
> export ANCHORE_CLI_USER=admin export ANCHORE_CLI_PASS=foobar
>
> \# 查看安装状态 anchore-cli \--u admin \--p foobar system status
>
> \# 数据库同步状态 anchore-cli \--u admin \--p foobar system feeds list
>
> \# 等待同步完成，添加镜像到anchore engine进行分析 anchore-cli \--u
> admin \--p foobar image add docker.io/library/nginx:latest
>
> \# 查看进度 anchore-cli \--u admin \--p foobar image list
>
> \# 等待分析完成，查看详情 anchore-cli \--u admin \--p foobar image get
> docker.io/library/nginx:latest
>
> \# 查看漏洞 anchore-cli \--u admin \--p foobar image vuln
> docker.io/library/nginx:latest os
>
> \# 添加仓库 anchore-cli repo add localhost:5000 \--noautosubscribe
> \--lookuptag alpine

参考资料：
\[官方docs\](<https://docs.anchore.com/current/docs/engine/general/concepts/policy/>)
\[anchore-policy hub\](<https://github.com/anchore/hub>) \[How to
Install and use Anchore Container Image Security Scanner? include
jenkins
plugin\](<https://geekflare.com/anchore-container-security-scanner/>)

## docker-registry

``` {.bash}
# Start your registry

docker run -d -p 5000:5000 --name registry registry:2

# Pull (or build) some image from the hub

docker pull ubuntu

# Tag the image so that it points to your registry

docker image tag ubuntu localhost:5000/myfirstimage

# Push it

docker push localhost:5000/myfirstimage

# Pull it back

docker pull localhost:5000/myfirstimage

# Now stop your registry and remove all data

docker container stop registry && docker container rm -v registry
```

## Jenkins

``` {.bash}
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins

systemctl start jenkins
```

打开浏览器访问：http://你的IP:8080，浏览器会提示输入管理员密码，可以从日志中查看到：\`cat
/var/log/jenkins/jenkins.log\`，输入后按照提示操作即可。

## gitlab

``` sh
version: '3.2'
services:

  gitlab:
    image: gitlab/gitlab-ce:latest
    hostname: 172.28.128.5
    container_name: gitlab
    restart: always
    volumes:
      - /gitlab/config:/etc/gitlab
      - /gitlab/logs:/var/log/gitlab
      - /gitlab/data:/var/opt/gitlab
    ports:
      - '443:443'
      - '80:80'
      - '2222:22'       

#将以上内容保存为docker-compose.yaml，然后在同一目录下运行`docker-compose up -d` ,访问http:你的IP 即可。
```

# 安装docker registry并使用anchore扫描

## 安装docker registry

1.  编写docker-compose.yml文件,volumes下的目录根据实际情况调整。

``` {.bash}
restart: always
image: registry:2
ports:
  - 5000:5000
environment:
  REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
  REGISTRY_HTTP_TLS_KEY: /certs/domain.key
  REGISTRY_AUTH: htpasswd
  REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
  REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
volumes:
  - /home/kali/registry/data:/var/lib/registry
  - /home/kali/registry/certs:/certs
  - /home/kali/registry/auth:/auth
```

2.  启动容器

```{=html}
<!-- -->
```
    docker-compose up -d

3.  添加insecure-registries

``` {.bash}
# 打开daemon.json
sudo vi /etc/docker/daemon.json

# 加入insecure-registries
{  
   "insecure-registries":["私库地址", "127.0.0.1:5000"]
}
```

4.  上传镜像

``` {.bash}
docker pull ubuntu
docker tag ubuntu:latest IP:5000/test:latest
docker push IP:5000/test:latest
```

5.  web管理

此时，访问https://IP:5000/v2/_catalog即可看到上传的镜像

a\) 列出存储库(数量n=99) :

    http://10.114.27.47:5000/v2/_catalog?n=99

b\) 查看镜像标签 :

    http://10.114.27.47:5000/v2/镜像名/tags/list

c\) 拉取镜像配置 :

    http://docker.io/v2/framework/<imageName>/manifests/latest<Reference>  //reference可以是标签或者digest

d\) 按层拉取镜像 :

    http://docker.io/v2/<image>/blobs/<digest> //digest是镜像每个fsLayer层的唯一标识。存在于上一步下载的配置的fsLayers里面。

## 配置证书和密码（可选）

**证书设置为可选设置**

``` {.bash}
# 创建工作目录,certs存放证书，auth存放用户名密码文件。后续需要映射到容器中。
mkdir data certs auth

# 创建账号密码testuser/testpassword
docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd
```

1.  生成CA证书

``` {.bash}
# 生成ca根证书秘钥文件ca.key
openssl genrsa -out "ca.key" 4096
# 生成根证书签发申请文件ca.csr
openssl req \
          -new -key "ca.key" \
          -out "ca.csr" -sha256 \
          -subj '/C=US/ST=CA/L=San Francisco/O=Docker/CN=Swarm Secret Example CA'
# 创建根证书配置文件，命名为ca.cnf
  [root_ca]
  basicConstraints = critical,CA:TRUE,pathlen:1
  keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
  subjectKeyIdentifier=hash
# 签发根证书文件，生成ca.crt
openssl x509 -req  -days 3650  -in "ca.csr" \
               -signkey "ca.key" -sha256 -out "ca.crt" \
               -extfile "ca.cnf" -extensions \
               root_ca
```

2.  通过CA签发站点证书

``` {.bash}
# 生成一个私钥site.key
openssl genrsa -out "site.key" 4096
# 生成服务器证书签证申请文件site.csr
openssl req -new -key "site.key" -out "site.csr" -sha256 \
      -subj '/C=US/ST=CA/L=San Francisco/O=Docker/CN=localhost'

    # 划重点： 命令中的localhost需要替换为将来用于访问仓库的域名或IP，例如：registry.local.com或者ip: 172.28.128.5。

# 创建配置文件site.cnf
[server]
authorityKeyIdentifier=keyid,issuer
basicConstraints = critical,CA:FALSE
extendedKeyUsage=serverAuth
keyUsage = critical, digitalSignature, keyEncipherment
subjectAltName = DNS:localhost, IP:127.0.0.1
subjectKeyIdentifier=hash

    # 划重点： localhost和127.0.0.1要换为仓库的访问域名和ip地址，例如：registry.local.com和
    # 192.168.10.100。

# 用根证书签证服务器证书，生成site.crt证书文件
openssl x509 -req -days 750 -in "site.csr" -sha256 \
-CA "ca.crt" -CAkey "ca.key"  -CAcreateserial \
-out "site.crt" -extfile "site.cnf" -extensions server
```

\[参考链接\](<https://www.jianshu.com/p/46c34e80a57b>)

3.  信任签发的CA

添加CA到系统根证书

``` {.bash}
# 添加证书
sudo cp site.crt /usr/local/share/ca-certificates
sudo update-ca-certificates

# 删除证书
sudo rm -f /usr/local/share/ca-certificates/site.crt  
sudo update-ca-certificates
```

## 使用anchore-cli扫描仓库

**容器部署请注意容器IP和宿主机IP的访问地址，可以通过\`ip
addr\`查看容器的地址，如：172.18.0.1**

> ``` {.bash}
> # 添加registry,对于需要登录的registry，必须要先添加到registry，保存用户/密码，anchore engine扫描的时候会使用registry中保存的信息pull镜像。
> anchore-cli registry add 172.18.0.1:5000 --insecure testuser testpassword
>
> # http协议使用以下命令：加上--skip-validate
> anchore-cli registry add 10.114.27.47:5000 --insecure --skip-validate . .
>
> # 添加repo(需要首先添加registry，再添加repo)，repo为/v2/_catalog下看到的内容
> anchore-cli repo add 172.18.0.1:5000/test
> ```

\[How to Install and use Anchore Container Image Security Scanner?
\](<https://geekflare.com/anchore-container-security-scanner/>)

# cicd测试

## 创建 Dockerfile

``` {.sh}
# 创建Dockerfile文件输入以下内容
FROM nginx
RUN echo 'This is a image build test - nginx' > /usr/share/nginx/html/index.html

# 在 Dockerfile 文件的存放目录下，执行构建动作
docker build -t nginx:test .

# 运行测试
dokcer run --name test -p 8181:80 nginx:test
```

## 创建构建任务(freestyle)

1.  源码管理添加Git Repo

2.  系统管理\--系统配置\--anchore plugin添加如下配置 :

        url：http://172.28.128.5:8228/v1
        user：admin
        passwd:foobar

3.  构建\--选择执行shell，添加以下内容

``` {.bash}
# 定义变量
APP_NAME="demo"
APP_VERSION="0.0.1"
APP_PORT=8181
IMAGE_NAME="172.28.128.5:5000/test/$APP_NAME:$BUILD_NUMBER"
CONTAINER_NAME=$APP_NAME-$APP_VERSION

# 进入target 目录复制Dockerfile 文件
# cd $WORKSPACE/target
# cp classes/Dockerfile .

#构建docker 镜像
docker build -t $IMAGE_NAME .

# 登录
docker login 172.28.128.5:5000 -u testuser -p testpassword

#推送docker镜像
docker push $IMAGE_NAME

# Line added to create anchore_images file
echo "$IMAGE_NAME $WORKSPACE/Dockerfile " > anchore_images

#删除同名docker容器
cid=$(docker ps -a| grep "$CONTAINER_NAME" | awk '{print $1}')
if [ "$cid" != "" ]; then
   docker rm -f $cid
fi

#启动docker 容器
docker run -d -p $APP_PORT:80 --name $CONTAINER_NAME $IMAGE_NAME

#删除 Dockerfile 文件
# rm -f Dockerfile
```

## CICD过程中如果遇到提示：docker.sock: connect: permission denied

``` {.bash}
# 添加jenkins用户到docker组
sudo usermod -a -G docker jenkins

# 重启Jenkins服务
systemctl restart jenkins
```

\[jenkins anchore
plugin\](<https://github.com/jenkinsci/anchore-container-scanner-plugin>)
\[来源于菜鸟教程\](<https://www.runoob.com/docker/docker-dockerfile.html>)

\[anchore-engine\](<https://github.com/anchore/anchore-engine>)
\[Anchore Container Image Scanner
Plugin\](<https://github.com/jenkinsci/anchore-container-scanner-plugin>)
\[anchore-cli\](<https://github.com/anchore/anchore-cli>)
\[官方参考文档\](<https://docs.anchore.com/current/docs/using/cli_usage/repositories/>)

# clair

安装clair扫描器并使用clairctl扫描

## clair安装

1\. 修改config.yml :

    source: postgresql://postgres:111@localhost:5432?sslmode=disable

## 参考链接

[76%都存在漏洞？！Docker镜像安全扫描应该这样做](https://developer.aliyun.com/article/661987)
:

    https://developer.aliyun.com/article/661987

[clair客户端衍生工具](https://github.com/quay/clair/blob/master/Documentation/integrations.md)
:

    https://github.com/quay/clair/blob/master/Documentation/integrations.md
