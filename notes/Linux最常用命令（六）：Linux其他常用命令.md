# Linux最常用命令（六）：Linux其他常用命令

- 清屏命令：```clear```
- 查询命令详细参数命令：```man```
- 挂载命令：```mnt```
- 远程连接服务 SSH 相关命令：
  - 启动 SSH 服务命令：```service sshd start```
  - 重启 SSH 服务命令：```service sshd restart```
  - 关闭 SSH 服务命令：```service sshd stop```

Linux 大多数情况下都是远程服务器，开发者通过远程工具连接 Linux ，启动了某个项目的 JAR，一旦窗口关闭，JAR 也就停止运行了，因此一般通过如下命令启动 JAR：

```bash
nohup java -jar jar-0.0.1-SNAPSHOT.jar &
```

这里多了 nohup ，表示当前窗口关闭时服务不挂起，继续在后台运行。