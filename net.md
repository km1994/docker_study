# 【关于 docker 安装】那些你不知道的事

> 作者：杨夕
> 
> docker 项目地址：https://github.com/km1994/docker_study
> 
> 论文学习项目地址：https://github.com/km1994/nlp_paper_study
> 
> 大佬们好，我叫杨夕，该项目主要是本人在研读顶会论文和复现经典论文过程中，所见、所思、所想、所闻，可能存在一些理解错误，希望大佬们多多指正。
> 
> NLP 面经地址：https://github.com/km1994/NLP-Interview-Notes
> 
> 大佬们好，我叫杨夕，该项目主要是本人在找工作过程中，面试准备和经历，可能存在一些理解错误，希望大佬们多多指正。

- [【关于 docker 安装】那些你不知道的事](#关于-docker-安装那些你不知道的事)
  - [一、什么是 Docker 基础网络？](#一什么是-docker-基础网络)
    - [1.1 外部如何访问容器？](#11-外部如何访问容器)
    - [1.2 映射所有接口地址](#12-映射所有接口地址)
    - [1.3 映射到指定地址的指定端口](#13-映射到指定地址的指定端口)
    - [1.4 映射到指定地址的任意端口](#14-映射到指定地址的任意端口)
    - [1.5 映射到指定地址的任意端口](#15-映射到指定地址的任意端口)
  - [参考资料](#参考资料)

## 一、什么是 Docker 基础网络？

### 1.1 外部如何访问容器？

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过-P或-p参数来指定端口映射。

当使用-P标记时，Docker会随机映射一个端口到内部容器开放的网络端口。 使用docker container ls可以看到，本地主机的 32768 被映射到了容器的 80 端口。此时访问本机的 32768 端口即可访问容器内 NGINX 默认页面。

```s
$ docker run -d -P nginx:alpine

$ docker container ls -l
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
fae320d08268        nginx:alpine        "/docker-entrypoint.…"   24 seconds ago      Up 20 seconds       0.0.0.0:32768->80/tcp   bold_mcnulty
```

同样的，可以通过docker logs命令来查看访问记录。
```s
$ docker logs fa
172.17.0.1 - - [25/Aug/2020:08:34:04 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:80.0) Gecko/20100101 Firefox/80.0" "-"
```
-p则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort.

### 1.2 映射所有接口地址

使用hostPort:containerPort格式本地的 80 端口映射到容器的 80 端口，可以执行

```s
$ docker run -d -p 80:80 nginx:alpine
```

此时默认会绑定本地所有接口上的所有地址。

### 1.3 映射到指定地址的指定端口

可以使用ip:hostPort:containerPort格式指定映射使用一个特定地址，比如localhost地址127.0.0.1

```s
$ docker run -d -p 127.0.0.1:80:80 nginx:alpine
```

### 1.4 映射到指定地址的任意端口

使用ip::containerPort绑定localhost的任意端口到容器的80端口，本地主机会自动分配一个端口。

```s
$ docker run -d -p 127.0.0.1::80 nginx:alpine
```

还可以使用udp标记来指定udp端口

```s
$ docker run -d -p 127.0.0.1:80:80/udp nginx:alpine
```

### 1.5 映射到指定地址的任意端口

使用docker port来查看当前映射的端口配置，也可以查看到绑定的地址

```s
$ docker port fa 80
0.0.0.0:32768
```

注意： 容器有自己的内部网络和 ip 地址（使用docker inspect查看，Docker还可以有一个可变的网络配置。） -p标记可以多次使用来绑定多个端口

例如

```s
$ docker run -d     -p 80:80     -p 443:443     nginx:alpine
```

待续！！！

## 参考资料

1. [Docker简介](https://github.com/datawhalechina/team-learning-program/blob/master/Docker/04%20Docker网络.md)