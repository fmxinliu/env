## 部署 SVN+SSH

#### 1. 安装ssh server和subversion
```sh
yum install -y openssh-server subversion
```

#### 2. 新建系统用户svnuser，此用户用于ssh登录帐号
```sh
useradd svnuser
```

#### 3. 给svnuser添加svn访问权限
```sh
vim /opt/svn/test/conf/authz
```

#### 4. 新建系统用户dev，此用户用于开发账号
```sh
useradd dev
echo dev | passwd --stdin dev
```

#### 5. 在客户端，为dev创建ssh密钥
```sh
su dev
cd
ssh-keygen -t rsa -b 1024 -C "dev"
```

#### 6. 在服务器端，为dev创建authorized_keys
- 拷贝dev公钥
```sh
su
mkdir /home/svnuser/.ssh
cp -p /home/dev/.ssh/id_rsa.pub /home/svnuser/.ssh/id_rsa_dev.pub
```
- 生成authorized_keys
```sh
cd /home/svnuser/.ssh
echo -n 'command="/usr/bin/svnserve -t -r /opt/svn --listen-port 3690 --tunnel-user=svnuser",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_dev.pub >> authorized_keys
```
- 修改ssh目录权限
```sh
chmod 700 /home/svnuser/.ssh
chmod 600 /home/svnuser/.ssh/authorized_keys
chown -R svnuser:svnuser /home/svnuser/.ssh
```

#### 7. 在客户端，配置ssh
- 在用户目录生成.subversion
```sh
su dev
cd
svn co
```
- 拷贝dev私钥
```sh
cp -p /home/dev/.ssh/id_rsa /home/dev/.subversion/id_rsa_dev
```
- 配置ssh：在`[tunnels]`内增加一行
```sh
vim /home/dev/.subversion/config
ssh = /usr/bin/ssh -l svn -i /home/dev/.subversion/id_rsa_dev
```

#### 8. 测试ssh免密登录
```sh
svn co svn+ssh://svnuser@192.168.244.250/test/trunk test
```