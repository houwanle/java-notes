# Linux最常用命令（五）：Linux进程管理

在 Linux 的应用中，我们需要对进程进行管理，如查看某个进程是否启动、以及在必要的时刻，杀掉某个线程。

## 查看进程命令

```bash
ps
```

ps 命令是 Linux 操作系统中查看进程的命令，通过 ps 命令我们可以查看 Linux 操作系统中正在运行的过程，并可以获得进程的 PID（进程的唯一标识），通过 PID 可以对进程进行相应的管理。

```bash
ps -ef | grep [进程关键字]
```

根据进程关键词查看进程命令显示如下，显示的进程列表中第一列表示开启进程的用户，第二列表示进程唯一标识 PID，第三列表示父进程 PPID，第四列表示 CPU 占用资源比列，最后一列表示进程所执行程序的具体位置。

```bash
[shang@localhost ~]$ ps -ef|grep sshd
root 1829 1  0 May24 ?   00:00:00 /usr/sbin/sshd
shang 24166 24100  0   20:17 pts/2  00:00:00      grep  sshd
[shang@localhost ~]$
```

## 杀掉进程命令

```bash
kill
```

当系统中有进程进入死循环，或者需要被关闭时，我们可以使用 kill 命令对其关闭。

```kill -9 [PID]``` PID 为 Linux 操作系统中进程的标识