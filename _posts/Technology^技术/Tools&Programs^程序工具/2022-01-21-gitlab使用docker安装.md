---
title: gitlab使用docker安装
author: Narule
date: 2022-01-21 18:00:00 +0800
categories: [Technology^技术, Tools&Programs^程序工具]
tags: [writing, linux, docker]
---

# gitlab 使用docker安装

## 环境

centos7

内存最低不小于4G

## 安装docker

[前文](https://www.cnblogs.com/Narule/p/14461433.html)



### 清理 卸载之前的docker环境



```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
```



### 安装工具包设置存储库

```bash
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```



### 安装docker程序

```bash
yum list docker-ce --showduplicates | sort -r

yum install docker-ce-18.09.0 docker-ce-cli-18.09.0  containerd.io

# 启动
systemctl start docker
```



## 下载gitlab镜像

```bash
# 搜索镜像
docker search gitlab

# 下载镜像
docker pull gitlab/gitlab-ce

# 下载完成之后 docker iamges  命令可以查看
[root@localhost ~]# docker images 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              5b4c624c7fe1        33 hours ago        519MB
gitlab/gitlab-ce    latest              f9cc225c75e1        3 days ago          2.36GB


```





## 启动镜像

```bash
docker run \
 -itd  \
 --publish 9443:443 --publish 11180:80 --publish 11922:22 \
 -v /usr/local/gitlab/etc:/etc/gitlab  \
 -v /usr/local/gitlab/log:/var/log/gitlab \
 -v /usr/local/gitlab/opt:/var/opt/gitlab \
 --restart always \
 --privileged=true \
 --name gitlab \
 gitlab/gitlab-ce
```

上面的命令主要意思的文件和端口映射关系配置

访问主机的11180端口，会被转发到gitlab容器的80端口

访问主机的11922端口，会被转发到gitlab容器的22端口

容器的名称叫 gitlab ,后面停止运行容器 输入 docker stop gitlab即可

## 配置

启动后可以通过 docker ps 查看启动的容器

启动后过一会，通过浏览器输入 http://IP:11180 即可访问容器gitlab

通过

```bash
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```
命令我们可以拿到gitlab root账号的初始密码登录即可

访问地址配置，虽然容器启动，但是项目拉取和访问生成的地址是容器的id,

### 访问地址的修改
可能是容器版本的问题，在项目访问上需要额外做一些配置，gitlab才能正常使用
比如主机的地址是:192.168.11.22
gitlab容器id是:ff3dgts
项目名称是:code 
项目的访问地址生成的是 http://ff3dgts/code 这样导致git客户端不能成功拉去代码，

我们应该改成 http://192.168.11.22:11180/code 这样才能生效成功访问代码

```bash
# 进入容器
docker exec -it gitlab  /bin/bash

# 修改配置
vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml


#找到下面的配置修改

  gitlab:
    ## Web server settings (note: host is the FQDN, do not include http://)
	host: 192.168.11.22 #改为主机地址
    port: 11180 #改为主机端口
	https: false
	
  gitlab_shell：
	ssh_host: 192.168.11.22  #改为主机地址

	ssh_port: 11922   #改为主机端口

```

### 重启容器

在gitlab容器里面:

```bash
gitlab-ctl restart 
```
