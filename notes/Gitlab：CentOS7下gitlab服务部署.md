## CentOS7下gitlab服务部署

### 1.下载 Gitlab
- 下载地址：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/
- 这里以gitlab-ce-10.8.2-ce.0.el7.x86_64.rpm为例；

### 2. 安装
将gitlab安装包上传到CentOS7中的 /opt 中，并新建install.sh 文件，内容如下：
```bash
sudo rpm -ivh /opt/gitlab-ce-10.8.2-ce.0.el7.x86_64.rpm
sudo yum install -y curl policycoreutils-python openssh-server cronie
sudo lokkit -s http -s ssh
sudo yum install postfix
sudo service postfix start
sudo chkconfig postfix on
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="http://gitlab.example.com" yum -y install gitlab-ce
```

执行以下命令：
```bash
#设置权限
chmod 755 install.sh

#执行install.sh文件
cd /opt
./install.sh

#重启linux
reboot

#初始化gitlab配置
gitlab-ctl reconfigure

#启动gitlab服务
gitlab-ctl start

#停止gitlab服务
gitlab-ctl stop
```
