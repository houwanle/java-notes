## 常用的Shell脚本

### 1. 数据库相关Shell脚本

- MySQL数据库备份单循环

```bash
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
HOST=localhost
USER=backup
PASS=123.com
BACKUP_DIR=/data/db_backup
DB_LIST=$(mysql -h$HOST -u$USER -p$PASS -s -e "show databases;" 2>/dev/null |egrep -v "Database|information_schema|mysql|performance_schema|sys")

for DB in $DB_LIST; do
    BACKUP_NAME=$BACKUP_DIR/${DB}_${DATE}.sql
    if ! mysqldump -h$HOST -u$USER -p$PASS -B $DB > $BACKUP_NAME 2>/dev/null; then
        echo "$BACKUP_NAME 备份失败!"
    fi
done
```

---

- MySQL数据库备份多循环

```bash
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
HOST=localhost
USER=backup
PASS=123.com
BACKUP_DIR=/data/db_backup
DB_LIST=$(mysql -h$HOST -u$USER -p$PASS -s -e "show databases;" 2>/dev/null |egrep -v "Database|information_schema|mysql|performance_schema|sys")

for DB in $DB_LIST; do
    BACKUP_DB_DIR=$BACKUP_DIR/${DB}_${DATE}
    [ ! -d $BACKUP_DB_DIR ] && mkdir -p $BACKUP_DB_DIR &>/dev/null
    TABLE_LIST=$(mysql -h$HOST -u$USER -p$PASS -s -e "use $DB;show tables;" 2>/dev/null)
    for TABLE in $TABLE_LIST; do
        BACKUP_NAME=$BACKUP_DB_DIR/${TABLE}.sql
        if ! mysqldump -h$HOST -u$USER -p$PASS $DB $TABLE > $BACKUP_NAME 2>/dev/null; then
            echo "$BACKUP_NAME 备份失败!"
        fi
    done
done
```

---

### 2. Nginx相关的Shell脚本

- Nginx访问日志按天切割

```bash
#!/bin/bash
LOG_DIR=/usr/local/nginx/logs
YESTERDAY_TIME=$(date -d "yesterday" +%F)
LOG_MONTH_DIR=$LOG_DIR/$(date +"%Y-%m")
LOG_FILE_LIST="default.access.log"

for LOG_FILE in $LOG_FILE_LIST; do
    [ ! -d $LOG_MONTH_DIR ] && mkdir -p $LOG_MONTH_DIR
    mv $LOG_DIR/$LOG_FILE $LOG_MONTH_DIR/${LOG_FILE}_${YESTERDAY_TIME}
done

kill -USR1 $(cat /var/run/nginx.pid)
```

---

- Nginx访问日志分析脚本

```bash
#!/bin/bash
# 日志格式: $remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"
LOG_FILE=$1
echo "统计访问最多的10个IP"
awk '{a[$1]++}END{print "UV:",length(a);for(v in a)print v,a[v]}' $LOG_FILE |sort -k2 -nr |head -10
echo "----------------------"

echo "统计时间段访问最多的IP"
awk '$4>="[01/Dec/2018:13:20:25" && $4<="[27/Nov/2018:16:20:49"{a[$1]++}END{for(v in a)print v,a[v]}' $LOG_FILE |sort -k2 -nr|head -10
echo "----------------------"

echo "统计访问最多的10个页面"
awk '{a[$7]++}END{print "PV:",length(a);for(v in a){if(a[v]>10)print v,a[v]}}' $LOG_FILE |sort -k2 -nr
echo "----------------------"

echo "统计访问页面状态码数量"
awk '{a[$7" "$9]++}END{for(v in a){if(a[v]>5)print v,a[v]}}'
```

---



- Dos攻击防范（自动屏蔽攻击 IP）

```bash
#!/bin/bash
DATE=$(date +%d/%b/%Y:%H:%M)
LOG_FILE=/usr/local/nginx/logs/demo2.access.log
ABNORMAL_IP=$(tail -n5000 $LOG_FILE |grep $DATE |awk '{a[$1]++}END{for(i in a)if(a[i]>10)print i}')
for IP in $ABNORMAL_IP; do
    if [ $(iptables -vnL |grep -c "$IP") -eq 0 ]; then
        iptables -I INPUT -s $IP -j DROP
        echo "$(date +'%F_%T') $IP" >> /tmp/drop_ip.log
    fi
done
```

---

- Linux系统发送告警脚本

```bash
# yum install mailx
# vi /etc/mail.rc
set from=baojingtongzhi@163.com smtp=smtp.163.com
set smtp-auth-user=baojingtongzhi@163.com smtp-auth-password=123456
set smtp-auth=login
```


---



- 查看网卡实时流量脚本

```bash
#!/bin/bash
NIC=$1
echo -e " In ------ Out"
while true; do
    OLD_IN=$(awk '$0~"'$NIC'"{print $2}' /proc/net/dev)
    OLD_OUT=$(awk '$0~"'$NIC'"{print $10}' /proc/net/dev)
    sleep 1
    NEW_IN=$(awk  '$0~"'$NIC'"{print $2}' /proc/net/dev)
    NEW_OUT=$(awk '$0~"'$NIC'"{print $10}' /proc/net/dev)
    IN=$(printf "%.1f%s" "$((($NEW_IN-$OLD_IN)/1024))" "KB/s")
    OUT=$(printf "%.1f%s" "$((($NEW_OUT-$OLD_OUT)/1024))" "KB/s")
    echo "$IN $OUT"
    sleep 1
done
```

---

- 服务器系统配置初始化脚本

```bash
#/bin/bash
# 设置时区并同步时间
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
if ! crontab -l |grep ntpdate &>/dev/null ; then
    (echo "* 1 * * * ntpdate time.windows.com >/dev/null 2>&1";crontab -l) |crontab
fi

# 禁用selinux
sed -i '/SELINUX/{s/permissive/disabled/}' /etc/selinux/config

# 关闭防火墙
if egrep "7.[0-9]" /etc/redhat-release &>/dev/null; then
    systemctl stop firewalld
    systemctl disable firewalld
elif egrep "6.[0-9]" /etc/redhat-release &>/dev/null; then
    service iptables stop
    chkconfig iptables off
fi

# 历史命令显示操作时间
if ! grep HISTTIMEFORMAT /etc/bashrc; then
    echo 'export HISTTIMEFORMAT="%F %T `whoami` "' >> /etc/bashrc
fi

# SSH超时时间
if ! grep "TMOUT=600" /etc/profile &>/dev/null; then
    echo "export TMOUT=600" >> /etc/profile
fi

# 禁止root远程登录
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# 禁止定时任务向发送邮件
sed -i 's/^MAILTO=root/MAILTO=""/' /etc/crontab

# 设置最大打开文件数
if ! grep "* soft nofile 65535" /etc/security/limits.conf &>/dev/null; then
    cat >> /etc/security/limits.conf << EOF
    * soft nofile 65535
    * hard nofile 65535
EOF
fi

# 系统内核优化
cat >> /etc/sysctl.conf << EOF
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_tw_buckets = 20480
net.ipv4.tcp_max_syn_backlog = 20480
net.core.netdev_max_backlog = 262144
net.ipv4.tcp_fin_timeout = 20
EOF

# 减少SWAP使用
echo "0" > /proc/sys/vm/swappiness

# 安装系统性能分析工具及其他
yum install gcc make autoconf vim sysstat net-tools iostat if
```

---

- 监控100台服务器磁盘利用率脚本

```bash
#!/bin/bash
HOST_INFO=host.info
for IP in $(awk '/^[^#]/{print $1}' $HOST_INFO); do
    USER=$(awk -v ip=$IP 'ip==$1{print $2}' $HOST_INFO)
    PORT=$(awk -v ip=$IP 'ip==$1{print $3}' $HOST_INFO)
    TMP_FILE=/tmp/disk.tmp
    ssh -p $PORT $USER@$IP 'df -h' > $TMP_FILE
    USE_RATE_LIST=$(awk 'BEGIN{OFS="="}/^\/dev/{print $NF,int($5)}' $TMP_FILE)
    for USE_RATE in $USE_RATE_LIST; do
        PART_NAME=${USE_RATE%=*}
        USE_RATE=${USE_RATE#*=}
        if [ $USE_RATE -ge 80 ]; then
            echo "Warning: $PART_NAME Partition usage $USE_RATE%!"
        fi
    done
done
```

---

- 监控内存和磁盘容量，小于给定值时报警

```bash
#!/bin/bash

# 实时监控本机内存和硬盘剩余空间,剩余内存小于500M、根分区剩余空间小于1000M时,发送报警邮件给root管理员

# 提取根分区剩余空间
disk_size=$(df / | awk '/\//{print $4}')

# 提取内存剩余空间
mem_size=$(free | awk '/Mem/{print $4}')
while :
do
# 注意内存和磁盘提取的空间大小都是以 Kb 为单位
if  [  $disk_size -le 512000 -a $mem_size -le 1024000  ]
then
    mail  ‐s  "Warning"  root  <<EOF
  Insufficient resources,资源不足
EOF
fi
done
```

---

- 编写helloworld脚本

```bash
#!/bin/bash
# 编写hello world脚本
echo "Hello World!"
```

---

- 通过位置变量创建Linux系统账户及密码

```bash
#!/bin/bash
# 通过位置变量创建 Linux 系统账户及密码
#$1 是执行脚本的第一个参数,$2 是执行脚本的第二个参数
useradd    "$1"
echo "$2"  |  passwd  ‐‐stdin  "$1"
```

---

- 备份日志

```bash
#!/bin/bash
# 每周 5 使用 tar 命令备份/var/log 下的所有日志文件
# vim  /root/logbak.sh
# 编写备份脚本,备份后的文件名包含日期标签,防止后面的备份将前面的备份数据覆盖
# 注意 date 命令需要使用反引号括起来,反引号在键盘<tab>键上面
tar  -czf  log-`date +%Y%m%d`.tar.gz  /var/log

# crontab ‐e  #编写计划任务,执行备份脚本
00  03  *  *  5  /root/logbak.sh
```

---

- 一键部署LNMP(RPM 包版本)

```bash
#!/bin/bash
# 一键部署 LNMP(RPM 包版本)
# 使用 yum 安装部署 LNMP,需要提前配置好 yum 源,否则该脚本会失败
# 本脚本使用于 centos7.2 或 RHEL7.2
yum ‐y install httpd
yum ‐y install mariadb mariadb‐devel mariadb‐server
yum ‐y install php  php‐mysql

systemctl start httpd mariadb
systemctl enable httpd mariadb
```

---

- 猜数字游戏

```bash
#!/bin/bash

# 脚本生成一个 100 以内的随机数,提示用户猜数字,根据用户的输入,提示用户猜对了,
# 猜小了或猜大了,直至用户猜对脚本结束。
 
# RANDOM 为系统自带的系统变量,值为 0‐32767的随机数
# 使用取余算法将随机数变为 1‐100 的随机数
num=$[RANDOM%100+1]
echo "$num"
 
# 使用 read 提示用户猜数字
# 使用 if 判断用户猜数字的大小关系:‐eq(等于),‐ne(不等于),‐gt(大于),‐ge(大于等于),
# ‐lt(小于),‐le(小于等于)
while  :
do
  read -p "计算机生成了一个 1‐100 的随机数,你猜: " cai
    if [ $cai -eq $num ]
    then
         echo "恭喜,猜对了"
         exit
      elif [ $cai -gt $num ]
      then
             echo "Oops,猜大了"
        else
             echo "Oops,猜小了"
   fi
done
```

---

- 检测本机当前用户是否为超级管理员,如果是管理员,则使用 yum 安装 vsftpd,如果不是,则提示您非管理员(使用字串对比版本)

```bash
#!/bin/bash

# 检测本机当前用户是否为超级管理员,如果是管理员,则使用 yum 安装 vsftpd,如果不
# 是,则提示您非管理员(使用字串对比版本) 
if [ $USER == "root" ]
then
  yum ‐y install vsftpd
else
    echo "您不是管理员,没有权限安装软件"
fi
```

---

- 检测本机当前用户是否为超级管理员,如果是管理员,则使用 yum 安装 vsftpd,如果不是,则提示您非管理员(使用 UID 数字对比版本)

```bash
#!/bin/bash

# 检测本机当前用户是否为超级管理员,如果是管理员,则使用 yum 安装 vsftpd,如果不
# 是,则提示您非管理员(使用 UID 数字对比版本)
if [ $UID -eq 0 ];then
    yum ‐y install vsftpd
else
    echo "您不是管理员,没有权限安装软件"
fi
```

---

- 编写脚本:提示用户输入用户名和密码,脚本自动创建相应的账户及配置密码。如果用户不输入账户名,则提示必须输入账户名并退出脚本;如果用户不输入密码,则统一使用默认的 123456 作为默认密码。

```bash
#!/bin/bash

# 编写脚本:提示用户输入用户名和密码,脚本自动创建相应的账户及配置密码。如果用户
# 不输入账户名,则提示必须输入账户名并退出脚本;如果用户不输入密码,则统一使用默
# 认的 123456 作为默认密码。
 
read -p "请输入用户名: " user
#使用‐z 可以判断一个变量是否为空,如果为空,提示用户必须输入账户名,并退出脚本,退出码为 2
#没有输入用户名脚本退出后,使用$?查看的返回码为 2
if [ -z $user ];then
     echo "您不需输入账户名"
   exit 2
fi
#使用 stty ‐echo 关闭 shell 的回显功能
#使用 stty  echo 打开 shell 的回显功能
stty -echo
read -p "请输入密码: " pass
stty echo
pass=${pass:‐123456}
useradd "$user"
echo "$pass" | passwd ‐‐stdin "$user"
```

---

- 输入三个数并进行升序排序

```bash
#!/bin/bash

# 依次提示用户输入 3 个整数,脚本根据数字大小依次排序输出 3 个数字
read -p "请输入一个整数:" num1
read -p "请输入一个整数:" num2
read -p "请输入一个整数:" num3
# 不管谁大谁小,最后都打印 echo "$num1,$num2,$num3"
# num1 中永远存最小的值,num2 中永远存中间值,num3 永远存最大值
# 如果输入的不是这样的顺序,则改变数的存储顺序,如:可以将 num1 和 num2 的值对调
tmp=0
# 如果 num1 大于 num2,就把 num1 和和 num2 的值对调,确保 num1 变量中存的是最小值
if [ $num1 -gt $num2 ];then   
  tmp=$num1
  num1=$num2
  num2=$tmp
fi
# 如果 num1 大于 num3,就把 num1 和 num3 对调,确保 num1 变量中存的是最小值
if [ $num1 -gt $num3 ];then   
    tmp=$num1
    num1=$num3
    num3=$tmp
fi
# 如果 num2 大于 num3,就把 num2 和 num3 对标,确保 num2 变量中存的是小一点的值
if [ $num2 -gt $num3 ];then
    tmp=$num2
    num2=$num3
    num3=$tmp
fi
echo "排序后数据(从小到大)为:$num1,$num2,$num3"
```

---

- 石头、剪刀、布游戏

```bash
#!/bin/bash

# 编写脚本,实现人机<石头,剪刀,布>游戏
game=(石头 剪刀 布)
num=$[RANDOM%3]
computer=${game[$num]}
# 通过随机数获取计算机的出拳
# 出拳的可能性保存在一个数组中,game[0],game[1],game[2]分别是 3 中不同的可能
 
echo "请根据下列提示选择您的出拳手势"
echo "1.石头"
echo "2.剪刀"
echo "3.布"
 
read -p "请选择 1‐3:" person
case  $person  in
1)
  if [ $num -eq 0 ]
  then
    echo "平局"
    elif [ $num -eq 1 ]
    then
      echo "你赢"
  else
    echo "计算机赢"
  fi;;
2)   
  if [ $num -eq 0 ]
  then
    echo "计算机赢"
    elif [ $num -eq 1 ]
    then
      echo "平局"
  else
    echo "你赢"
  fi;;
3)
  if [ $num -eq 0 ]
  then
    echo "你赢"
    elif [ $num -eq 1 ]
    then
      echo "计算机赢"
  else
    echo "平局"
  fi;;
*)
  echo "必须输入 1‐3 的数字"
esac
```

---

- 编写脚本测试 192.168.4.0/24 整个网段中哪些主机处于开机状态,哪些主机处于关机状态(for 版本)

```bash
#!/bin/bash

# 编写脚本测试 192.168.4.0/24 整个网段中哪些主机处于开机状态,哪些主机处于关机
# 状态(for 版本)
for i in {1..254}
do
  # 每隔0.3秒ping一次，一共ping2次，并以1毫秒为单位设置ping的超时时间
     ping ‐c 2 ‐i 0.3 ‐W 1 192.168.4.$i  &>/dev/null
    if  [ $? -eq 0 ];then
         echo "192.168.4.$i is up"
     else
         echo  "192.168.4.$i is down"
     fi
done
```

---

- 编写脚本测试 192.168.4.0/24 整个网段中哪些主机处于开机状态,哪些主机处于关机状态(while 版本)

```bash
#!/bin/bash

# 编写脚本测试 192.168.4.0/24 整个网段中哪些主机处于开机状态,哪些主机处于关机
# 状态(while 版本) 
i=1
while [ $i -le 254 ]
do
     ping ‐c 2 ‐i 0.3 ‐W 1 192.168.4.$i  &>/dev/null
     if  [ $? -eq 0 ];then
         echo "192.168.4.$i is up"
    else
         echo  "192.168.4.$i is down"
     fi
     let i++
done
```

---

- 编写脚本测试 192.168.4.0/24 整个网段中哪些主机处于开机状态,哪些主机处于关机状态(多进程版)

```bash
#!/bin/bash

# 编写脚本测试 192.168.4.0/24 整个网段中哪些主机处于开机状态,哪些主机处于关机
# 状态(多进程版)
 
#定义一个函数,ping 某一台主机,并检测主机的存活状态
myping(){
ping ‐c 2 ‐i 0.3 ‐W 1 $1  &>/dev/null
if  [ $? -eq 0 ];then
  echo "$1 is up"
else
  echo "$1 is down"
fi
}
for i in {1..254}
do
     myping 192.168.4.$i &
done
# 使用&符号,将执行的函数放入后台执行
# 这样做的好处是不需要等待ping第一台主机的回应,就可以继续并发ping第二台主机,依次类推。
```

---

- 编写脚本,显示进度条

```bash
#!/bin/bash

# 编写脚本,显示进度条
jindu(){
while :
do
     echo -n '#'
     sleep 0.2
done
}
jindu &
cp -a $1 $2
killall $0
echo "拷贝完成"
```

---

- 进度条,动态时针版本；定义一个显示进度的函数,屏幕快速显示|  / ‐ \

```bash
#!/bin/bash

# 进度条,动态时针版本
# 定义一个显示进度的函数,屏幕快速显示|  / ‐ \
rotate_line(){
INTERVAL=0.5  #设置间隔时间
COUNT="0"     #设置4个形状的编号,默认编号为 0(不代表任何图像)
while :
do
  COUNT=`expr $COUNT + 1` #执行循环,COUNT 每次循环加 1,(分别代表4种不同的形状)
  case $COUNT in          #判断 COUNT 的值,值不一样显示的形状就不一样
  "1")                    #值为 1 显示‐
          echo -e '‐'"\b\c"
          sleep $INTERVAL
          ;;
    "2")                  #值为 2 显示\\,第一个\是转义
          echo -e '\\'"\b\c"
          sleep $INTERVAL
          ;;
    "3")                  #值为 3 显示|
          echo -e "|\b\c"
          sleep $INTERVAL
          ;;
   "4")                   #值为 4 显示/
          echo -e "/\b\c"
          sleep $INTERVAL
          ;;
    *)                    #值为其他时,将 COUNT 重置为 0
          COUNT="0";;
    esac
done
}
rotate_line
```

---

- 9*9 乘法表

```bash
#!/bin/bash

# 9*9 乘法表(编写 shell 脚本,打印 9*9 乘法表) 
for i in `seq 9`
do
    for j in `seq $i`
     do
         echo -n "$j*$i=$[i*j]  "
     done
    echo
done
```

---

- 使用死循环实时显示 eth0 网卡发送的数据包流量

```bash
#!/bin/bash

# 使用死循环实时显示 eth0 网卡发送的数据包流量 
 
while :
do
   echo  '本地网卡 eth0 流量信息如下: '
    ifconfig eth0 | grep "RX pack" | awk '{print $5}'
    ifconfig eth0 | grep "TX pack" | awk '{print $5}'
     sleep 1
done
```

---

- 使用 user.txt 文件中的人员名单,在计算机中自动创建对应的账户并配置初始密码本脚本执行,需要提前准备一个 user.txt 文件,该文件中包含有若干用户名信息

```bash
#!/bin/bash

# 使用 user.txt 文件中的人员名单,在计算机中自动创建对应的账户并配置初始密码
# 本脚本执行,需要提前准备一个 user.txt 文件,该文件中包含有若干用户名信息
for i in `cat user.txt`
do
     useradd  $i
     echo "123456" | passwd ‐‐stdin $i
done
```

---

- 编写批量修改扩展名脚本

```bash
#!/bin/bash

# 编写批量修改扩展名脚本,如批量将 txt 文件修改为 doc 文件 
# 执行脚本时,需要给脚本添加位置参数
# 脚本名  txt  doc(可以将 txt 的扩展名修改为 doc)
# 脚本名  doc  jpg(可以将 doc 的扩展名修改为 jpg)
 
for i in `ls *.$1`
do
     mv $i ${i%.*}.$2
done
```

---

- 使用 expect 工具自动交互密码远程其他主机安装 httpd 软件

```bash
#!/bin/bash

# 使用 expect 工具自动交互密码远程其他主机安装 httpd 软件 
 
# 删除~/.ssh/known_hosts 后,ssh 远程任何主机都会询问是否确认要连接该主机
rm  ‐rf  ~/.ssh/known_hosts
expect <<EOF
spawn ssh 192.168.4.254
expect "yes/no" {send "yes\r"}
# 根据自己的实际情况将密码修改为真实的密码字串
expect "password" {send  "密码\r"}
expect "#" {send  "yum ‐y install httpd\r"}
expect "#" {send  "exit\r"}
EOF
```

---

- 一键部署 LNMP(源码安装版本)

```bash
#!/bin/bash

# 一键部署 LNMP(源码安装版本)
menu()
{
clear
echo "  ##############‐‐‐‐Menu‐‐‐‐##############"
echo "# 1. Install Nginx"
echo "# 2. Install MySQL"
echo "# 3. Install PHP"
echo "# 4. Exit Program"
echo "  ########################################"
}
 
choice()
{
  read -p "Please choice a menu[1‐9]:" select
}
 
install_nginx()
{
  id nginx &>/dev/null
  if [ $? -ne 0 ];then
    useradd -s /sbin/nologin nginx
  fi
  if [ -f nginx‐1.8.0.tar.gz ];then
    tar -xf nginx‐1.8.0.tar.gz
    cd nginx‐1.8.0
    yum -y install  gcc pcre‐devel openssl‐devel zlib‐devel make
    ./configure ‐‐prefix=/usr/local/nginx ‐‐with‐http_ssl_module
    make
    make install
    ln -s /usr/local/nginx/sbin/nginx /usr/sbin/
    cd ..
  else
    echo "没有 Nginx 源码包"
  fi
}
 
install_mysql()
{
  yum -y install gcc gcc‐c++ cmake ncurses‐devel perl
  id mysql &>/dev/null
  if [ $? -ne 0 ];then
    useradd -s /sbin/nologin mysql
  fi
  if [ -f mysql‐5.6.25.tar.gz ];then
    tar -xf mysql‐5.6.25.tar.gz
    cd mysql‐5.6.25
    cmake .
    make
    make install
    /usr/local/mysql/scripts/mysql_install_db ‐‐user=mysql ‐‐datadir=/usr/local/mysql/data/
‐‐basedir=/usr/local/mysql/
    chown -R root.mysql /usr/local/mysql
    chown -R mysql /usr/local/mysql/data
    /bin/cp -f /usr/local/mysql/support‐files/mysql.server /etc/init.d/mysqld
    chmod +x /etc/init.d/mysqld
    /bin/cp -f /usr/local/mysql/support‐files/my‐default.cnf /etc/my.cnf
    echo "/usr/local/mysql/lib/" >> /etc/ld.so.conf
    ldconfig
    echo 'PATH=\$PATH:/usr/local/mysql/bin/' >> /etc/profile
    export PATH
  else
    echo  "没有 mysql 源码包"
    exit
  fi
}
 
install_php()
{
#安装 php 时没有指定启动哪些模块功能,如果的用户可以根据实际情况自行添加额外功能如‐‐with‐gd 等
yum  -y  install  gcc  libxml2‐devel
if [ -f mhash‐0.9.9.9.tar.gz ];then
  tar -xf mhash‐0.9.9.9.tar.gz
  cd mhash‐0.9.9.9
  ./configure
  make
  make install
  cd ..
if [ ! ‐f /usr/lib/libmhash.so ];then
  ln -s /usr/local/lib/libmhash.so /usr/lib/
fi
ldconfig
else
  echo "没有 mhash 源码包文件"
  exit
fi
if [ -f libmcrypt‐2.5.8.tar.gz ];then
  tar -xf libmcrypt‐2.5.8.tar.gz
  cd libmcrypt‐2.5.8
  ./configure
  make
  make install
  cd ..
  if [ ! -f /usr/lib/libmcrypt.so ];then  
    ln -s /usr/local/lib/libmcrypt.so /usr/lib/
  fi
  ldconfig
else
  echo "没有 libmcrypt 源码包文件"
  exit
fi
if [ -f php‐5.4.24.tar.gz ];then
  tar -xf php‐5.4.24.tar.gz
  cd php‐5.4.24
  ./configure  ‐‐prefix=/usr/local/php5  ‐‐with‐mysql=/usr/local/mysql  ‐‐enable‐fpm    ‐‐
  enable‐mbstring  ‐‐with‐mcrypt  ‐‐with‐mhash  ‐‐with‐config‐file‐path=/usr/local/php5/etc  ‐‐with‐
  mysqli=/usr/local/mysql/bin/mysql_config
  make && make install
  /bin/cp -f php.ini‐production /usr/local/php5/etc/php.ini
  /bin/cp -f /usr/local/php5/etc/php‐fpm.conf.default /usr/local/php5/etc/php‐fpm.conf
  cd ..
else
  echo "没有 php 源码包文件"
  exit
fi 
}
 
while :
do
  menu
  choice
  case $select in
  1)
    install_nginx
    ;;
  2)
    install_mysql
    ;;
  3)
    install_php
    ;;
  4)
    exit
    ;;
  *)
    echo Sorry!
  esac
done
```

---

- 编写脚本快速克隆 KVM 虚拟机

```bash
#!/bin/bash

# 编写脚本快速克隆 KVM 虚拟机
 
# 本脚本针对 RHEL7.2 或 Centos7.2
# 本脚本需要提前准备一个 qcow2 格式的虚拟机模板,
# 名称为/var/lib/libvirt/images  /.rh7_template 的虚拟机模板
# 该脚本使用 qemu‐img 命令快速创建快照虚拟机
# 脚本使用 sed 修改模板虚拟机的配置文件,将虚拟机名称、UUID、磁盘文件名、MAC 地址
# exit code:  
#    65 ‐> user input nothing
#    66 ‐> user input is not a number
#    67 ‐> user input out of range
#    68 ‐> vm disk image exists
 
IMG_DIR=/var/lib/libvirt/images
BASEVM=rh7_template
read -p "Enter VM number: " VMNUM
if [ $VMNUM -le 9 ];then
VMNUM=0$VMNUM
fi
 
if [ -z "${VMNUM}" ]; then
    echo "You must input a number."
    exit 65
elif [[  ${VMNUM} =~ [a‐z]  ]; then
    echo "You must input a number."
    exit 66
elif [ ${VMNUM} -lt 1 -o ${VMNUM} -gt 99 ]; then
    echo "Input out of range"
    exit 67
fi
 
NEWVM=rh7_node${VMNUM}
 
if [ -e $IMG_DIR/${NEWVM}.img ]; then
    echo "File exists."
    exit 68
fi
 
echo -en "Creating Virtual Machine disk image......\t"
qemu‐img create -f qcow2 ‐b $IMG_DIR/.${BASEVM}.img $IMG_DIR/${NEWVM}.img &> /dev/null
 
echo -e "\e[32;1m[OK]\e[0m"
 
#virsh dumpxml ${BASEVM} > /tmp/myvm.xml
cat /var/lib/libvirt/images/.rhel7.xml > /tmp/myvm.xml
sed -i "/<name>${BASEVM}/s/${BASEVM}/${NEWVM}/" /tmp/myvm.xml
sed -i "/uuid/s/<uuid>.*<\/uuid>/<uuid>$(uuidgen)<\/uuid>/" /tmp/myvm.xml
sed -i "/${BASEVM}\.img/s/${BASEVM}/${NEWVM}/" /tmp/myvm.xml
 
# 修改 MAC 地址,本例使用的是常量,每位使用该脚本的用户需要根据实际情况修改这些值 
# 最好这里可以使用便利,这样更适合于批量操作,可以克隆更多虚拟机 
sed -i "/mac /s/a1/0c/" /tmp/myvm.xml
 
echo -en "Defining new virtual machine......\t\t"
virsh define /tmp/myvm.xml &> /dev/null
echo -e "\e[32;1m[OK]\e[0m"
```

---

- 点名器脚本

```bash
#!/bin/bash

# 编写一个点名器脚本
 
# 该脚本,需要提前准备一个 user.txt 文件
# 该文件中需要包含所有姓名的信息,一行一个姓名,脚本每次随机显示一个姓名
while :
do
#统计 user 文件中有多少用户
line=`cat user.txt |wc ‐l`
num=$[RANDOM%line+1]
sed -n "${num}p"  user.txt
sleep 0.2
clear
done
```

---

- 查看有多少远程的 IP 在连接本机

```bash
#!/bin/bash

# 查看有多少远程的 IP 在连接本机(不管是通过 ssh 还是 web 还是 ftp 都统计) 
 
# 使用 netstat ‐atn 可以查看本机所有连接的状态,‐a 查看所有,
# -t仅显示 tcp 连接的信息,‐n 数字格式显示
# Local Address(第四列是本机的 IP 和端口信息)
# Foreign Address(第五列是远程主机的 IP 和端口信息)
# 使用 awk 命令仅显示第 5 列数据,再显示第 1 列 IP 地址的信息
# sort 可以按数字大小排序,最后使用 uniq 将多余重复的删除,并统计重复的次数
netstat -atn  |  awk  '{print $5}'  | awk  '{print $1}' | sort -nr  |  uniq -c
```

---

- 对 100 以内的所有正整数相加求和(1+2+3+4...+100)

```bash
#!/bin/bash

# 对 100 以内的所有正整数相加求和(1+2+3+4...+100)
 
#seq 100 可以快速自动生成 100 个整数
sum=0
for i in `seq 100`
do
    sum=$[sum+i]
done
echo "总和是:$sum"
```

---

- 统计 13:30 到 14:30 所有访问 apache 服务器的请求有多少个

```bash
#!/bin/bash

# 统计 13:30 到 14:30 所有访问 apache 服务器的请求有多少个
 
# awk 使用‐F 选项指定文件内容的分隔符是/或者:
# 条件判断$7:$8 大于等于 13:30,并且要求,$7:$8 小于等于 14:30
# 最后使用 wc ‐l 统计这样的数据有多少行,即多少个
awk -F "[ /:]" '$7":"$8>="13:30" && $7":"$8<="14:30"' /var/log/httpd/access_log |wc -l
```

---

- 统计 13:30 到 14:30 所有访问本机 Aapche 服务器的远程 IP 地址是什么

```bash
#!/bin/bash

# 统计 13:30 到 14:30 所有访问本机 Aapche 服务器的远程 IP 地址是什么 
# awk 使用‐F 选项指定文件内容的分隔符是/或者:
# 条件判断$7:$8 大于等于 13:30,并且要求,$7:$8 小于等于 14:30
# 日志文档内容里面,第 1 列是远程主机的 IP 地址,使用 awk 单独显示第 1 列即可
awk -F "[ /:]" '$7":"$8>="13:30" && $7":"$8<="14:30"{print $1}' /var/log/httpd/access_log
```

---

- 打印国际象棋棋盘

```bash
#!/bin/bash

# 打印国际象棋棋盘
# 设置两个变量,i 和 j,一个代表行,一个代表列,国际象棋为 8*8 棋盘
# i=1 是代表准备打印第一行棋盘,第 1 行棋盘有灰色和蓝色间隔输出,总共为 8 列
# i=1,j=1 代表第 1 行的第 1 列;i=2,j=3 代表第 2 行的第 3 列
# 棋盘的规律是 i+j 如果是偶数,就打印蓝色色块,如果是奇数就打印灰色色块
# 使用 echo ‐ne 打印色块,并且打印完成色块后不自动换行,在同一行继续输出其他色块
for i in {1..8}
do
    for j in {1..8}
    do
      sum=$[i+j]
    if [  $[sum%2] -eq 0 ];then
       echo -ne "\033[46m  \033[0m"
    else
      echo -ne "\033[47m  \033[0m"
    fi
    done
    echo
done
```

---

- 统计每个远程 IP 访问了本机 apache 几次?

```bash
#!/bin/bash

# 统计每个远程 IP 访问了本机 apache 几次? 
awk  '{ip[$1]++}END{for(i in ip){print ip[i],i}}'  /var/log/httpd/access_log
```

---

- 统计当前 Linux 系统中可以登录计算机的账户有多少个

```bash
#!/bin/bash

# 统计当前 Linux 系统中可以登录计算机的账户有多少个
#方法 1:
grep "bash$" /etc/passwd | wc -l
#方法 2:
awk -f: '/bash$/{x++}end{print x}'  /etc/passwd
```

---

- 统计/var/log 有多少个文件,并显示这些文件名

```bash
#!/bin/bash

# 统计/var/log 有多少个文件,并显示这些文件名 
# 使用 ls 递归显示所有,再判断是否为文件,如果是文件则计数器加 1
cd  /var/log
sum=0
for i in `ls -r *`
do
   if [ -f $i ];then
       let sum++
         echo "文件名:$i"
     fi
done
echo "总文件数量为:$sum"
```

---

- 自动为其他脚本添加解释器信息

```bash
#!/bin/bash

# 自动为其他脚本添加解释器信息#!/bin/bash,如脚本名为 test.sh 则效果如下: 
# ./test.sh  abc.sh  自动为 abc.sh 添加解释器信息
# ./test.sh  user.sh  自动为 user.sh 添加解释器信息
 
# 先使用 grep 判断对象脚本是否已经有解释器信息,如果没有则使用 sed 添加解释器以及描述信息
if  !  grep  -q  "^#!"  $1; then
sed  '1i #!/bin/bash'  $1
sed  '2i #Description: '
fi
# 因为每个脚本的功能不同,作用不同,所以在给对象脚本添加完解释器信息,以及 Description 后还希望
# 继续编辑具体的脚本功能的描述信息,这里直接使用 vim 把对象脚本打开,并且光标跳转到该文件的第 2 行
vim +2 $1
```

---

- 自动化部署 varnish 源码包软件

```bash
#!/bin/bash

# 自动化部署 varnish 源码包软件 
# 本脚本需要提前下载 varnish‐3.0.6.tar.gz 这样一个源码包软件,该脚本即可用自动源码安装部署软件
 
yum -y install gcc readline‐devel pcre‐devel
useradd -s /sbin/nologin varnish
tar -xf varnish‐3.0.6.tar.gz
cd varnish‐3.0.6
 
# 使用 configure,make,make install 源码安装软件包
./configure ‐‐prefix=/usr/local/varnish
make && make install
 
# 在源码包目录下,将相应的配置文件拷贝到 Linux 系统文件系统中
# 默认安装完成后,不会自动拷贝或安装配置文件到 Linux 系统,所以需要手动 cp 复制配置文件
# 并使用 uuidgen 生成一个随机密钥的配置文件
 
cp redhat/varnish.initrc /etc/init.d/varnish
cp redhat/varnish.sysconfig /etc/sysconfig/varnish
cp redhat/varnish_reload_vcl /usr/bin/
ln -s /usr/local/varnish/sbin/varnishd /usr/sbin/
ln -s /usr/local/varnish/bin/* /usr/bin
mkdir /etc/varnish
cp /usr/local/varnish/etc/varnish/default.vcl /etc/varnish/
uuidgen > /etc/varnish/secret
```

---

- 编写nginx启动脚本

```bash
#!/bin/bash

# 编写 nginx 启动脚本 
# 本脚本编写完成后,放置在/etc/init.d/目录下,就可以被 Linux 系统自动识别到该脚本
# 如果本脚本名为/etc/init.d/nginx,则 service nginx start 就可以启动该服务
# service nginx stop 就可以关闭服务
# service nginx restart 可以重启服务
# service nginx status 可以查看服务状态
 
program=/usr/local/nginx/sbin/nginx
pid=/usr/local/nginx/logs/nginx.pid
start(){
if [ -f $pid ];then
  echo  "nginx 服务已经处于开启状态"
else
  $program
fi
stop(){
if [ -! -f $pid ];then
  echo "nginx 服务已经关闭"
else
  $program -s stop
  echo "关闭服务 ok"
fi
}
status(){
if [ -f $pid ];then
  echo "服务正在运行..."
else
  echo "服务已经关闭"
fi
}
 
case $1 in
start)
  start;;
stop)
  stop;;
restart)
  stop
  sleep 1
  start;;
status)
  status;;
*)
  echo  "你输入的语法格式错误"
esac
```

---

- 自动对磁盘分区、格式化、挂载

```bash
 #!/bin/bash
 
# 自动对磁盘分区、格式化、挂载
# 对虚拟机的 vdb 磁盘进行分区格式化,使用<<将需要的分区指令导入给程序 fdisk
# n(新建分区),p(创建主分区),1(分区编号为 1),两个空白行(两个回车,相当于将整个磁盘分一个区)
# 注意:1 后面的两个回车(空白行)是必须的!
fdisk /dev/vdb << EOF
n
p
1
 
 
wq
EOF
 
#格式化刚刚创建好的分区
mkfs.xfs   /dev/vdb1
 
#创建挂载点目录
if [ -e /data ]; then
exit
fi
mkdir /data
 
#自动挂载刚刚创建的分区,并设置开机自动挂载该分区
echo '/dev/vdb1     /data    xfs    defaults        1 2'  >> /etc/fstab
mount -a
```

---

- 自动优化Linux内核参数

```bash
#!/bin/bash

# 自动优化 Linux 内核参数
 
#脚本针对 RHEL7
cat >> /usr/lib/sysctl.d/00‐system.conf <<EOF
fs.file‐max=65535
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 5
net.ipv4.tcp_syn_retries = 5
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
#net.ipv4.tcp_keepalive_time = 120
net.ipv4.ip_local_port_range = 1024  65535
kernel.shmall = 2097152
kernel.shmmax = 2147483648
kernel.shmmni = 4096
kernel.sem = 5010 641280 5010 128
net.core.wmem_default=262144
net.core.wmem_max=262144
net.core.rmem_default=4194304
net.core.rmem_max=4194304
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_keepalive_time = 30
net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_sack = 0
EOF
 
sysctl –p
```