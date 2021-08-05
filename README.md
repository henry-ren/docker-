docker使用
==============================

1.安装docker
------------------------------
```
sudo apt-get remove docker docker-engine docker.io  *卸载旧版本*
# 添加传输软件包与CA证书
sudo apt-get update
sudo apt-get install  apt-transport-https  ca-certificates  curl  \
gnupg-agent software-properties-common

# 添加docker国内源，加速下载
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

```
如果出现  curl: (77) error setting certificate verify locations

export CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
```

```
# sources.list中添加docker软件源
sudo add-apt-repository \
"deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
$(lsb_release -cs) stable"

# 安装docker
sudo apt-get update
sudo apt-get install docker.io

# 启动docker
sudo systemctl enable docker
sudo systemctl start docker
```
## 2.创建dockers用户组
-----------------------------------------------------
同样地，这一步也需要服务器的管理员进行操作。创建docker用户组的用途是为了避免服务器的普通用户使用docker过程中使用sudo权限，也避免root用户使用docker服务时频繁输入密码。
```
# 建立用户组
sudo groupadd docker
# 添加当前用户到用户组
sudo usermod -aG docker $USER

# 更新docker用户组
newgrp docker              
 
# 退出远程终端，并测试docker
docker run hello-world  # 自动下载并拉取hello-word镜像
```
## 3.拉取与查看镜像
--------------------------------------------------
当前，阿里云提供了以下基础镜像源，我们选择Pytorch-1.4-cuda10.1版本的镜像进行拉取即可。
```
# Python：
registry.cn-shanghai.aliyuncs.com/tcc-public/python:3

# Pytorch：
registry.cn-shanghai.aliyuncs.com/tcc-public/pytorch:latest-py3 
registry.cn-shanghai.aliyuncs.com/tcc-public/pytorch:latest-cuda9.0-py3  
registry.cn-shanghai.aliyuncs.com/tcc-public/pytorch:1.1.0-cuda10.0-py3
registry.cn-shanghai.aliyuncs.com/tcc-public/pytorch:1.4-cuda10.1-py3

# TensorFlow：
registry.cn-shanghai.aliyuncs.com/tcc-public/tensorflow:latest-py3
registry.cn-shanghai.aliyuncs.com/tcc-public/tensorflow:1.1.0-cuda8.0-py2
registry.cn-shanghai.aliyuncs.com/tcc-public/tensorflow:1.12.0-cuda9.0-py3
registry.cn-shanghai.aliyuncs.com/tcc-public/tensorflow:latest-cuda10.0-py3

# Keras：
registry.cn-shanghai.aliyuncs.com/tcc-public/keras:latest-py3
registry.cn-shanghai.aliyuncs.com/tcc-public/keras:latest-cuda9.0-py3
registry.cn-shanghai.aliyuncs.com/tcc-public/keras:latest-cuda10.0-py3

# 拉取tensorflow镜像
docker pull registry.cn-shanghai.aliyuncs.com/tcc-public/tensorflow:latest-cuda10.0-py3
```

查看当前环境的镜像列表
>docker images

## 4.启动容器
----------------------------------------------------
```
# 由于下载好的Tensorflow镜像名字过长，我们可以将其重命名
# docker image tag [IMAGE ID] [重命名：重命名TAG]
docker image tag 76c152fbfd03 tf-cuda10.0-py3

# 启动Tensorflow容器(不使用GPU)
# docker run -itd –name [启动的容器名字] [镜像名：镜像TAG] /bin/bash
docker run -itd --name tensor_ct tf-cuda10.0-py3 /bin/bash
```
```

# 所有GPU
docker run --gpus  all -itd --name tensor_ct tf-cuda10.0-py3 /bin/bash

# 指定GPU 1，运行容器
docker run --gpus  device=0 -itd --name tensor_ct tf-cuda10.0-py3 /bin/bash

# 查看容器,可以看到刚刚创建的torch_ct容器
docker ps  # 查看正在运行的容器
docker ps -a  # 查看所有容器，包含停止的容器
```

```
# 进入刚刚创建的tensor_ct容器
docker exec -it tensor_ct /bin/bash
# 在容器中创建data目录
mkdir data
ls
# 退出当前容器
exit
```

## 5.数据互传
-----------------------------------------------------
容器一般都与外界隔离。为此，容器与宿主机之间进行数据交互则需要使用docker技术。值得注意的是，容器与宿主机之间的数据交互都是要在宿主机上进行，为此，我们需要事先退出当前所在的容器。
```
# 复制本地文件到容器中
# docker cp 本地文件 容器:容器目录 
docker cp local_data tensor_ct:/root 

# 复制docker文件到本地
# docker cp 容器:容器文件路径 宿主机目录
docker cp trnsor_ct:/root/container_data  /home/rqe/docker_test
```

## 6.打包程序
--------------------------------------------------------
有时候，我们自己开发的软件需要被其他人所应用。为了减少代码环境配置的时间成本，我们可以将当前运行的容器进行打包，并输出镜像文件，传给使用者即可。
`没搞明白，先不写`

## 7.其他命令
```
# 启动/停止容器服务
docker start/stop  torch_ct
# 删除容器
docker rm [CONTAINER ID] 
# 删除镜像
# docker rmi [REPOSITORY:TAG]
docker rmi registry.cn-shanghai.aliyuncs.com/tcc-public/pytorch:1.4-cuda10.1-py3  # 删除下载好的阿里云镜像
```







