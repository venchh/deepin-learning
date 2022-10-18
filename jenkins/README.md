# jenkins

jenkins是一个开源自动化服务，有助于进行软件构建、测试和部署，促进持续集成和持续部署。

### Table of Contents

- [docker+jenkins部署](#docker+jenkins部署)
- [使用docker启动jenkins服务](#使用docker启动jenkins服务)
- [jenkins升级站点](#jenkins升级站点)
- [记一些docker操作](#记一些docker操作)


### docker+jenkins部署

```shell
$ sudo apt install docker   #安装docker
$ sudo docker pull jenkins/jenkins:lts-jdk11    #拉取jenkins镜像
```

### 使用docker启动jenkins服务

```shell
$ sudo docker run -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins:lts-jdk11
```

### jenkins升级站点

    https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/dynamic-stable-2.361.1/update-center.json

### 记一些docker操作

    docker volume ls        ## 查看卷
    docker volume rm ID     ## 删除卷,删除报错时使用下面的三个步骤
    docker ps -q -a         ## 查看所有docker容器
    docker rm ID            ## 删除容器后再删除卷
    docker volume rm ID     ## 删除卷

    docker ps -q -a                 ## 3ff996c5d60c为本地Jenkins容器
    docker start 3ff996c5d60c       ## 启动jenkins容器

