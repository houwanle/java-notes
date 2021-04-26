## CentOS7 系统语言切换
> 在CentOS系统中，中文语言界面虽然便于直观理解，但是经常要使用操作命令，遇到有中文目录的情况下，混杂有中英文名称，对输入字符和定位路径不太方便， 因此统一修改为全英文语言显示。

```bash
#查看当前系统的语言
locale

#修改locale.conf文件内容
cd /etc
vi locale.conf

#将系统语言设置成中文
LANG="zh_CN.UTF-8"

#将系统语言设置成英文
LANG="en_US.UTF-8"

#重启CentOS系统后即可生效
reboot
```
