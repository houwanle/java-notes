# Linux最常用命令（七）：Linux系统软件安装

## 常用软件安装

Linux 下常用的软件安装方式有3种。

- tar 安装：如果开发商提供的是 tar、tar.gz、tar.bz 格式的包（其中 tar 格式的为打包后没有压缩的包，gz 结尾的是按照 gzip 打包并压缩的软件包，tar.bz 是按照二进制方式打包并压缩的软件包），可以采用 tar 包安装，tar 安装方式本质上是解压软件开发商提供的软件包，之后在通过相应配置，完成软件的安装。

- rpm 安装：rpm 安装方式是 redhat Linux 系列推出的一个软件包管理器，类似于 Windows 下的 exe 安装程序，可以直接使用 rpm 命令安装。

- yum 安装：yum 安装本质上依然是 rpm 包安装，和 rpm 安装方式的不同之处是用户可以通过 yum 参数，指定安装的软件包，系统将自动从互联网上下载相应的 rpm 软件包。而无须用户关心软件包的下载地址，以及软件包的依赖关系。

## 软件安装常用命令

- 解压压缩命令：```tar```
- 语法：```tar [选项] [压缩包]```
- 解压 gzip 包：```tar -zxvf [包名]```
- 解压 bz 包：```tar -jxvf [包名]```
- 解压普通包：```tar -xvf [包名]```

取值 | 说明
---|---
-c | 指定特定目录压缩
-x | 从备份文件中还原文件
-t | 列出备份文件的内容
-r | 添加文件到已经压缩的文件
-z | 有 gzip 属性的（后缀是 gz 的）
-j | 有 bz2 属性的（后缀是 bz 的）
-z | 有 cpmpress 属性的
-v | 显示所有进程
-O | 将文件解压到标准输出
-f | 使用档案名称


## 安装卸载命令：rpm

- 语法：```rpm [选项] [软件包]```
- 查询是否已经安装了某软件包：```rpm -qa|grep [软件包关键词]```
- 卸载已经安装的软件包：```rpm -e 软件包全名```
- 安装软件包并查看进度：```rpm -ivh 软件包路径```

取值 | 说明
---|---
-ivh | 安装显示安装进度
-Uvh | 升级软件包
-qpl | 列出 rpm 软件包内的文件信息
-qpi | 列出 rpm 软件包的描述信息
-qf | 查找指定文件属于哪个 rpm 软件包
-Va | 检验所有的 rpm 软件包，查找丢失的文件
-e | 删除包
-qa | 查找已经安装的 rpm 包