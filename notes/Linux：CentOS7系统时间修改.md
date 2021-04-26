## CentOS7系统时间修改
```bash
# 查看系统时间
date

# 将CentOS系统日期设定成 2021年1月14日
date -s 01/14/21

# 将CentOS时间设定成19:45:00
date -s 19:45:00

# 同步系统时钟
clock -w

# 将当前时间和日期写入BIOS，避免重启失效
hwclock -w
```
