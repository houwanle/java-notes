## 查看Linux版本信息

### 查看linux内核版本
- 方法一
```bash
cat /proc/version
```

- 方法二
```bash
uname -a
```

### 查看Linux系统版本
- 方法一
```bash
#这个命令适用于所有的Linux发行版，包括RedHat、SUSE、DeBian...等发行版。
lsb_release -a
```

- 方法二
```bash
#此命令也适用于所有的Linux发行版。
cat /etc/issue
```

- 方法三
```bash
#这种方法只适合RedHat系列的Linux
cat /etc/redhat-release
```
