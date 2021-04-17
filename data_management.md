# 【关于 docker Docker Volume 】那些你不知道的事

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

## 一、引言

对于 Docker Volume，首先我们需要知道Docker的文件系统是如何工作的。**Docker镜像是由多个文件系统（只读层）叠加而成**。当我们启动一个容器的时候，Docker会加载只读镜像层并在其上（译者注：镜像栈顶部）添加一个读写层。如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏。当删除Docker容器，并通过该镜像重新启动时，之前的更改将会丢失。在Docker中，只读层及在顶部的读写层的组合被称为Union File System（联合文件系统）

## 二、什么是数据卷？

- 介绍：是一个**可供一个或多个容器使用的特殊目录**，它绕过 UFS (UNIX File System) ，可以提供很多有用的特性；
- 特性：
  - 数据卷可以在容器之间共享和重用；
  - 对数据卷的修改会立马生效；
  - 对数据卷的更新，不会影响镜像；
  - 数据卷默认会一直存在，即使容器被删除；

> 注：注意：数据卷的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）。

## 三、 如何创建一个数据卷？

### 3.1 创建 数据卷

```s
    $ docker volume create datawhale

    >>>
    datawhale
```

### 3.2 查看所有 数据卷

```s
    $ docker volume ls

    >>>
    DRIVER    VOLUME NAME
    local     64d54a73cf2ef569ec7001a20922fa76c19ed1cb74019eeee8cb45984b3e09d9
    local     datawhale
```

> 注：DRIVER 表示 设备，VOLUME NAME 表示 数据卷名称

### 3.3 查看 数据卷 信息

```s
    $ docker volume inspect datawhale

    >>>
    [
        {
            "CreatedAt": "2021-04-17T03:15:36Z",
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "/var/lib/docker/volumes/datawhale/_data",
            "Name": "datawhale",
            "Options": {},
            "Scope": "local"
        }
    ]
```

> 注：
> CreatedAt 表示 创建时间
> DRIVER 表示 设备
> VOLUME NAME 表示 数据卷名称
> Mountpoint 表示 对应的主机目录

## 四、 如何启动一个 挂载 数据卷 的 容器？

### 4.1 启动一个 挂载 数据卷 的 容器

在用 docker run 命令的时候，使用 --mount 标记来将数据卷挂载到容器里。在一次 docker run 中可以挂载多个 数据卷。

> 如下：创建一个名为 web 的容器，并加载一个数据卷到容器的 /usr/share/nginx/html 目录。

```s
    $ docker run -d -P \
    --name web \
    --mount source=datawhale,target=/usr/share/nginx/html \
    nginx:alpine

    >>>
    Unable to find image 'nginx:alpine' locally
    alpine: Pulling from library/nginx
    540db60ca938: Pull complete
    197dc8475a23: Pull complete
    39ea657007e5: Pull complete
    37afbf7d4c3d: Pull complete
    0c01f42c3df7: Pull complete
    d590d87c9181: Pull complete
    Digest: sha256:07ab71a2c8e4ecb19a5a5abcfb3a4f175946c001c8af288b1aa766d67b0d05d2
    Status: Downloaded newer image for nginx:alpine
    99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e
```

> 注：
> 
> name 名称
> 
> mount： source ：数据卷     target ：是容器内文件系统挂载点
> 
> 注意，可以不需要提前创建好数据卷，直接在运行容器的时候mount 这时如果不存在指定的数据卷，docker会自动创建，自动生成。

### 4.2 查看数据卷的具体信息

```s
docker inspect web
[
    {
        "Id": "99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e",
        "Created": "2021-04-17T03:32:08.8284856Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1170,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-04-17T03:32:09.4050303Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:a64a6e03b0551e1cefa94db6cc6677fb1efed3c557d173f79584ff4ec474b5ae",
        "ResolvConfPath": "/var/lib/docker/containers/99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e/hostname",
        "HostsPath": "/var/lib/docker/containers/99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e/hosts",
        "LogPath": "/var/lib/docker/containers/99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e/99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e-json.log",
        "Name": "/web",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": true,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                50,
                120
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "Mounts": [
                {
                    "Type": "volume",
                    "Source": "datawhale",
                    "Target": "/usr/share/nginx/html"
                }
            ],
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/6a450bbb89e23c4365d339cad2b8c67b4eeec38c71300281c08cc3eb10da9900-init/diff:/var/lib/docker/overlay2/eb5cd8af595ae38f569ddd8bd5ccb338f7fb29173bfa525e924a3ce8ee21ad1e/diff:/var/lib/docker/overlay2/621799bf15c6a69794ba572972ce9ef745313c67d50f67d745931dd5d0f745f0/diff:/var/lib/docker/overlay2/0d74552c124da114fdb364c492b4c661170bd296117a5fd0bdab2fcb6c8efed6/diff:/var/lib/docker/overlay2/c081cf8e604395fe900ca09e05d90b649754bd2bf4e529ec2294ab6478362e76/diff:/var/lib/docker/overlay2/ba0da6960cf461975fff7fe01ce3fac96f8c7fb25d76ad5b5065069d04c2f784/diff:/var/lib/docker/overlay2/9727e6f4156fdb96ce78aea639527ffe307a3b0c7eccad075a9dca0d77466602/diff",
                "MergedDir": "/var/lib/docker/overlay2/6a450bbb89e23c4365d339cad2b8c67b4eeec38c71300281c08cc3eb10da9900/merged",
                "UpperDir": "/var/lib/docker/overlay2/6a450bbb89e23c4365d339cad2b8c67b4eeec38c71300281c08cc3eb10da9900/diff",
                "WorkDir": "/var/lib/docker/overlay2/6a450bbb89e23c4365d339cad2b8c67b4eeec38c71300281c08cc3eb10da9900/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "datawhale",
                "Source": "/var/lib/docker/volumes/datawhale/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "99690bc8ef63",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.19.10",
                "NJS_VERSION=0.5.3",
                "PKG_RELEASE=1"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "nginx:alpine",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "cf2a19200a2e52e645214eeb348d83b2d337ab0d567a81b0963007efac006cc9",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "49153"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/cf2a19200a2e",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "13639560e1202968a8b74de82b5998979746d8b870d4b9b4bc5635146a8a5715",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "a3d055c82a526fb5f5554a6199955ee805d5229a63a2a0e9944e9ff42923f68e",
                    "EndpointID": "13639560e1202968a8b74de82b5998979746d8b870d4b9b4bc5635146a8a5715",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

> 注：数据卷 信息在 "Mounts" Key 下面

```s
    ...
    "Mounts": [
        {
            "Type": "volume",
            "Name": "datawhale",
            "Source": "/var/lib/docker/volumes/datawhale/_data",
            "Destination": "/usr/share/nginx/html",
            "Driver": "local",
            "Mode": "z",
            "RW": true,
            "Propagation": ""
        }
    ],
    ...
```

## 五、如何删除 数据卷？

### 5.1 操作

数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 数据卷。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 docker rm -v 这个命令。

```s
    $ docker ps
    >>>
    CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                   NAMES
    99690bc8ef63   nginx:alpine   "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes   0.0.0.0:49153->80/tcp   web

    $ docker rm -f 99690bc8ef63
    >>>
    99690bc8ef63

    $ docker volume rm datawhale  #datawhale为卷名
    >>>
    datawhale
```

由于 数据卷 挂载 在 容器上面，所以 需要将 容器 给停掉或删除，才能删除数据卷

无主的数据卷可能会占据很多空间，要清理请使用以下命令

```s
    $ docker volume prune
```

### 5.2 问题记录

直接 删除 数据卷 时 会报错
```s
    $ docker volume rm datawhale
    >>>
    Error response from daemon: remove datawhale: volume is in use - [99690bc8ef637866dca5800087e5fbb359c12640508215c2917dd133d013b75e]
```

## 六、挂载主机目录

### 6.1 挂载一个主机目录作为数据卷

使用 --mount 标记可以指定挂载一个本地主机的目录到容器中去。

```s
$ docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

上面的命令加载主机的 /src/webapp 目录到容器的 /usr/share/nginx/html目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用 -v 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 --mount 参数时如果本地目录不存在，Docker 会报错。

Docker 挂载主机目录的默认权限是 读写，用户也可以通过增加 readonly 指定为 只读。

注意： 如果挂载的目录不存在，创建容器时，docker 不会自动创建，此时会报错

```s
$ docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine
```

加了 readonly 之后，就挂载为 只读 了。如果你在容器内 /usr/share/nginx/html 目录新建文件，会显示如下错误

```s
/usr/share/nginx/html # touch new.txt
touch: new.txt: Read-only file system
```

### 6.2 查看数据卷的具体信息

在主机里使用以下命令可以查看 web 容器的信息

```s
$ docker inspect web
```

挂载主机目录 的配置信息在 "Mounts" Key 下面

```s
"Mounts": [
    {
        "Type": "bind",
        "Source": "/src/webapp",
        "Destination": "/usr/share/nginx/html",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

挂载一个本地主机文件作为数据卷

--mount 标记也可以从主机挂载单个文件到容器中

```s
$ docker run --rm -it \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
   bash
```

## 参考

1. [Docker 数据管理](https://vuepress.mirror.docker-practice.com/data_management/)