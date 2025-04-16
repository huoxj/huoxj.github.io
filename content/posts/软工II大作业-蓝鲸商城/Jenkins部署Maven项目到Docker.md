---
title: "Jenkins部署Maven项目到Docker"
date: 2024-05-10
series: 
- "软工II大作业-蓝鲸商城"
---

# 使用Jenkins部署Maven项目到Docker

> Jenkins：能够自动配置、自动部署项目。通过Webhook可以实现项目一旦被Push就会自动重新配置项目并且部署，以实现更新

## 1 准备步骤

### 1.1 安装Docker

这步比较简单，网上教程也很多，故省略

### 1.2 拉取jenkins镜像

执行拉取镜像的命令

```bash
docker pull jenkins/jenkins:lts
```

查看当前的镜像，可以发现jenkins已经在列表里了

```bash
docker images
```

```
REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
bluewhale-backend   master    044e4cb8919f   16 hours ago    318MB
mysql               latest    f3df03e3cfc9   8 days ago      585MB
jenkins/jenkins     lts       4e586344183a   3 weeks ago     469MB
openjdk             8-jre     0c14a0e20aa3   21 months ago   274MB
```

### 1.3 创建容器

使用**run**命令从镜像创建容器

**镜像和容器**的关系就像**类和对象的**关系

```bash
docker run -d --restart=always -uroot \
--name jenkins \
-p 8080:8080 \
-v /home/jenkins:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
-v ~/.ssh/:/root/.ssh \
jenkins/jenkins:lts
```

**参数说明**

- `-d`：让容器运行在后台，不占用当前的终端
- `--restart=always`：让docker重启后自动启动这个容器
- `-uroot`：容器以root身份创建
- `--name`：指定容器**名称**，否则docker会随机生成美丽的名字
- `-p`：将容器的端口**映射**到宿主机的端口。冒号前是宿主机的，后者是容器里的
- `-v`：将宿主机的目录**挂载**到容器里，能让容器访问宿主机的目录。冒号前宿主机，后容器。
    - 第一个挂载将宿主机的路径挂载到jenkins的工作目录，能让jenkins的工作目录持久化到宿主机，这样容器被销毁了也不会让jenkins的配置和数据被销毁
    - 第二个和第三个挂载是让容器内部能使用宿主机的docker命令，以便jenkins**在宿主机创建maven项目的容器**
    - 第四个挂载是把ssh的密钥挂载上，这样jenkins就可以用ssh从仓库拉取代码了。当然也可以在容器里面重新生成密钥对。

创建容器后**查看当前正在运行的容器**

```
docker ps
```

```bash
CONTAINER ID   IMAGE                 COMMAND                  CREATED              STATUS              PORTS                                                  NAMES
9ed58b36b0c4   jenkins/jenkins:lts   "/usr/bin/tini -- /u…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 50000/tcp   jenkins
```

排版很烂是因为这个表太长了，不过只要有jenkins就行

另外，使用`-a`选项可以**查看所有状态的容器**，包括已经退出和挂起的

**补充**：容器的生命周期(摘自CSDN[docker容器的生命周期-CSDN博客](https://blog.csdn.net/wBreak/article/details/134378761))

![img](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409272340788.png)

## 2 配置Jenkins

### 2.1 初始化Jenkins

#### 2.1.1 管理员初始密码

在浏览器访问8080端口进入Jenkins工作台

根据提示输入管理员密码

- 这里的路径是容器内的路径，所以要进入容器的命令行

- ```bash
    docker exec -it jenkins bash
    ```

我尝试过从宿主机挂载的目录访问，但是权限会有问题，~~懒得解决了~~

#### 2.1.2 安装插件

**安装推荐的插件**就行。

但是我的ECS内存太少了，所以我都是用什么插件装什么

#### 2.1.3 创建管理员账户

可以根据提示创建账户。如果不需要的话直接**使用admin账号继续**即可

#### 2.1.4 配置完成

一路点到配置完成，就进入Jenkins的Dashboard了

### 2.2 对Jenkins进行全局配置

#### 2.2.1 Dashboard->Manage Jenkins->Plugins

- 安装Maven集成插件，以便项目的构建

![image-20240510111502998](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409271711418.jpg)

#### 2.2.2 Dashboard->Manage Jenkins->Tools

- 安装JDK
    - Jenkins好像自带一个JDK。但是如果想用自己的版本的话就要在`JDK installations`里面配置自己的JDK
- 安装Maven
    - Maven可以自动安装，选择`Install automatically`就可以了

#### 2.2.3 Dashboard->Manage Jenkins->Credentials

一路点开System->Global credentials

根据自己的验证方式配置就行了。

我的是ssh密钥对，所以把`id_rsa`的内容**全部**(包括`-----BEGIN OPENSSH PRIVATE KEY-----`)复制进去

记得把公钥复制到仓库的凭据设置里面

## 3 部署Maven项目

