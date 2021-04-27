## Docker的常用命令
### 1. 帮助命令
```bash
#查看版本信息
docker version

#查看详细信息
docker info

#帮助命令
docker --help
```

### 2. 镜像命令
```bash
#查看本地镜像
docker images
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221195257879.png)

**说明：**
- REPOSITORY：表示镜像的仓库源；
- TAG：镜像的标签；
- IMAGE ID：镜像ID；
- CREATED：镜像创建时间；
- VIRTUAL：镜像大小；

> 同一个仓库源可以有多个TAG，代表这个仓库源的不同版本，我们使用REPOSITORY TAG 来定义不同的镜像。
> 如果你不指定一个镜像的版本标签，例如你只使用ubuntu，docker将默认使用 ubuntu:latest 镜像。

```bash
#列出本地所有的镜像（含中间映像层）
docker images -a

#只显示镜像ID
docker images -q

#显示镜像的摘要信息
docker images --digests

#显示完整的镜像信息
docker images --no-trunc
```

```bash
#搜索镜像（去 http://hub.docker.com 上搜索nginx镜像）
docker search nginx

#下载最新版本的nginx镜像
docker pull nginx
docker pull nginx:lastest

#删除单个镜像
docker rmi nginx
#强制删除镜像
docker rmi -f nginx
#删除多个镜像
docker rmi -f hello-world nginx
#删除全部镜像
docker rmi -f $(docker images -qa)
```

### 3. 容器命令
有镜像才能创建容器，这是根本前提。

```bash
#拉取（下载）centos镜像
docker pull centos
```
```bash
#新建启动容器命令
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

#案例
docker run -it --name mycentos1222 centos
```
OPTIONS说明：
- `--name="容器新名字"`：为容器指定一个名称
- `-d`：后台运行容器，并返回容器ID，也即启动守护式容器；
- `-i` ：以交互模式运行容器，通常与`-t`同时使用；
- `-t` ：为容器重新分配一个伪输入终端，通常与`-i`同时使用；
- `-P` ：随机端口映射；
- `-p` ： 指定端口映射，有以下四种格式：
  - `ip:hostPort:containerPort`
  - `ip:containerPort`
  - `hostPort:containerPort`
  - `containerPort`

```bash
#列出当前所有正在运行的容器+历史上运行过的
docker ps -a
#显示最近创建的容器
docker ps -l
#显示最近n个创建的容器
docker ps -n
#不截断输出
docker ps --no-trunc
```

退出容器，两种方式：
- `exit`：容器停止退出；
- `Ctrl + P + Q`：容器不停止退出；


```bash
#启动容器
docker start 容器ID或容器名称
#案例
docker start mycentos

#停止容器
docker stop 容器ID或容器名称
#案例
docker stop mycentos

#强制停止容器
docker kill 容器ID或容器名称

#重启容器
docker restart 容器ID或容器名称
#案例
docker restart mycentos

#删除已停止的容器
docker rm 容器ID
#强制删除容器
docker rm -f 容器ID

#一次删除多个容器
docker rm -f $(docker ps -a -q)
#或者
docker ps -a -q | xargs docker rm
```

```bash
#启动守护式进程（后台运行，但会自动退出）
docker run -d centos
```

查看容器日志
```bash
docker logs -f -t --tail 容器ID
```
说明：
- `-t`：加入时间戳；
- `-f`：跟随最新的日志打印
- `--tail`：数字显示最后多少条

```bash
#查看容器内运行的进程
docker top 容器ID

#查看容器内部细节
docker inspect 容器ID
```

进入正在运行的容器并以命令行交互
```bash
#是在容器中打开新的终端，并且可以启动新的进程
docker exec -it 容器ID bashShell

#直接进入容器启动命令的终端，不会启动新的进程
docker attach 容器ID
```

```bash
#从容器内拷贝文件到主机上
docker cp 容器ID:容器内路径 目的主机路径
```
