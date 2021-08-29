###  MySQL 服务

#### 1. 安装软件包

- 在线安装
```sh
# 配置yum源
yum -y install wget
wget http://repo.mysql.com/mysql57-community-release-el6-10.noarch.rpm
rpm -Uvh mysql57-community-release-el6-10.noarch.rpm

# 查看yum源
ls  /etc/yum.repos.d/ | grep mysql

# 查看可用的rpm包
yum repolist enabled | grep mysql

# 安装mysql server
yum -y install mysql-community-server

# 如果因数字证书验证失败，而导致安装失败：先卸载mysql数字证书，然后离线安装
rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n' | grep mysql
rpm -e gpg-pubkey-xxxxxxx-xxxxxxx
cd /var/cache/yum/x86_64/6/mysql57-community/packages
```

-  [离线安装](https://pan.baidu.com/disk/main?errmsg=Auth+Login+Sucess&errno=0&from=oldversion&ssnerror=0&#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fmysql)
```sh
# 手动安装
rpm -ivh mysql-community-common-5.7.35-1.el6.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.35-1.el6.x86_64.rpm
rpm -ivh mysql-community-client-5.7.35-1.el6.x86_64.rpm
rpm -ivh mysql-community-server-5.7.35-1.el6.x86_64.rpm

# 查看安装
rpm -qa | grep mysql

# 查看版本
mysql --version
```

#### 2. 配置服务

- 关闭 setlinux
```sh
# 临时关闭
setenforce 0

# 永久关闭
vi /etc/sysconfig/selinux
```

- 设置开机启动
```sh
# 设置开机启动
chkconfig mysqld on

# 查看设置是否成功
chkconfig --list | grep mysqld
```

- 开放端口
```sh
# 设置端口
vi /etc/my.cnf
# port = 3306

# 开放端口
vi /etc/sysconfig/iptables
# -A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```

- 启动服务
```sh
service mysqld start
```

#### 3. 配置数据库

- 设置 root 登录密码
```sh
grep 'temporary password' /var/log/mysqld.log
mysql -uroot -p"xxxxxxxxxxxx"
set global validate_password_policy=0;  # 只验证长度
set global validate_password_length=6;  # 修改密码长度，默认值是8个字符
alter user user()identified by"123456"; # 修改登陆密码
mysql -uroot -p123456
```

- 添加用户

```mysql
mysql> show databases;
mysql> use mysql;
mysql> show tables;
mysql> select user, authentication_string, host from user;
# mysql> update user set authentication_string=password('root') where user='root';
# mysql> flush privileges;

# 创建本地用户
mysql> create user 'user.local'@'localhost' identified by '123456';
mysql> grant all privileges on *.* to 'user.local'@'localhost';    # 授权所有权限
mysql> show grants for 'user.local'@'localhost';

# 创建不限主机登录用户
mysql> create user 'user.any'@'%' identified by '123456';
mysql> grant select, insert, update on *.* to 'user.any'@'%';     # 授权增、改、查权限
mysql> show grants for 'user.any'@'%';

mysql> quit
```
