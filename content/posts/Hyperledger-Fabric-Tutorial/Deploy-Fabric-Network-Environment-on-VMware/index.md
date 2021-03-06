---
title: "Deploy Fabric Network Environment on VMware"
date: 2020-11-14
draft: false
categories: [Blockchain]
tags: [Fabric]
description: 
featured_image:
author: 1uvu
---

{{< bilibili BV17K411V7qy>}}

[视频中的资料](https://github.com/1uvu/Blockchain-Notes/tree/master/fabric-notes/)

## 准备工作

### 虚拟机

1.  开启服务：程序与功能中开启 Hyper-V、虚拟机平台，并重启 Windows

2.  安装 VMware 和 Ubuntu 20.04 LTS Server

    1.  什么额外的软件都不要安
    2.  更换清华镜像：https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
    3.  取消更新并重启

3.  [设置VM静态IP](https://my.oschina.net/u/4271740/blog/4437308)

    1.  VMware 虚拟网卡 NAT 设置
    2.  Windows VMnet8 网卡 配置
    3.  VM 网络设置静态 IP

4.  设置VM自动启动

    1.  VMware 中共享 VM
    2.  共享 VM 开启自动启动和自动挂起

5.  关闭VM自动更新

    ```shell
    sudo nano /etc/apt/apt.conf.d/20auto-upgrades
    ```

### SSH

1.  设置 VM root 账户密码

    ```shell
    sudo passwd root
    ```

2. 设置 SSH

    1. 编辑 SSH 配置：

        ```shell
        sudo apt install ssh
        sudo nano /etc/ssh/sshd_config
        # ...
        Port 22
        # ...
        PermitRootLogin yes
        # ...
        PasswordAuthentication yes
        # ...
        ```

    2.  生成 SSH 密钥

        ```shell
        sudo ssh-keygen -A
        ```

    3.  重启 SSH 服务

        ```shell
        sudo service ssh start
        ```

3.  SSH 连接 root shell

    ```shell
    mkdir ~/Project
    ```

4. SSH 连接 ftp 上传部署需要的文件到 *Project* 目录

    

    >   fabric/
    >
    >   fabric-samples/
    >
    >   bootstrap.sh
    >
    >   go1.15.5.linux-amd64.tar.gz

### Docker

1.  添加 Docker 阿里云镜像加速

    打开阿里云[容器镜像服务页](https://cn.aliyun.com/product/acr)获取自己的加速地址，然后执行：

    ```shell
    mkdir -p /etc/docker
    tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://<你自己的ID>.mirror.aliyuncs.com"]
    }
    EOF
    ```

2.  安装 Docker

    ```shell
    # 将官方Docker库的GPG公钥添加到系统中
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    # 将Docker库添加到apt里
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
    # 再次更新下apt库列表
    apt update
    # 开始安装docker-ce
    apt install docker-ce
    
    # 安装完成查询版本号
    docker --version
    # 为 docker 添加当前用户
    service docker start
    usermod -aG docker $USER
    # 开始安装docker-compose
    apt install docker-compose
    
    # 安装完成后查询docker-compose版本号
    docker-compose --version
    ```

### Golang

1.  安装 Golang

    ```shell
    cd ~/Project
    tar -C /usr/local -xzf go1.15.5.linux-amd64.tar.gz
    ```

2.  配置 Golang 环境变量

    ```shell
    mkdir ~/GOPATH
    nano ~/.bashrc
    
    export GOROOT=/usr/local/go 
    export GOPATH=/root/GOPATH 
    export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
    
    source  ~/.bashrc
    ```

3.  Golang 镜像源加速

    ```shell
    go version
    go env -w GO111MODULE=on
    go env -w GOPROXY=https://goproxy.cn,direct
    ```

### 其他软件

```shell
apt install python3 python nodejs npm
```

## 部署 Fabric 环境

### 安装 Fabric

```shell
cd ~/Project
mkdir fabric
tar -zxvf hyperledger-fabric-ca-linux-amd64-1.4.9.tar.gz -C ./fabric
tar -zxvf hyperledger-fabric-linux-amd64-2.2.1.tar.gz -C ./fabric
mkdir /usr/local/fabric
mkdir /usr/local/fabric/bin
sudo cp fabric/bin/* /usr/local/fabric/bin
```

### 配置 Fabric 环境变量

```shell
nano ~/.bashrc 

export FABRIC_ROOT=/usr/local/fabric
export GOROOT=/usr/local/go # GOROOT是系统上安装 Go 软件包的位置。
export GOPATH=/root/GOPATH # GOPATH 是刚刚新建的工作目录的位置。
export PATH=$GOPATH/bin:$GOROOT/bin:$FABRIC_ROOT/bin:$PATH

source ~/.bashrc

# 执行以下不出错则配置成功
configtxgen version  
configtxlator version 
cryptogen version 
fabric-ca-client version 
fabric-ca-server version 
idemixgen version 
orderer version 
peer version
discover --help
```

### 安装 Docker 镜像

```shell
bash ./bootstrap.sh -s -b
```

### 环境测试

```shell
# git clone https://github.com/hyperledger/fabric-samples.git
cd ~/Project/fabric-samples
git switch -c v2.1.1
cp ../fabric/bin -r ./bin
cp ../fabric/config -r ./config
cd test-network/
service docker restart
chmod +x scripts/*	
apt install dos2unix
dos2unix network.sh
dos2unix scripts/*
dos2unix organizations/*
dos2unix organizations/fabric-ca/*
bash network.sh -h
bash network.sh up createChannel -ca -c mychannel -s couchdb -i 2.2.1
bash network.sh deployCC

export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
# 调用智能合约-已部署的 ChainCode，不出错则测试成功
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'

bash network.sh down
```

​	按照 [官方文档](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html) 进行测试