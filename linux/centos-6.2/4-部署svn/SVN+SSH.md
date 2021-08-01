## 部署 SVN+SSH

### 一、服务器安装和配置
#### 1. 安装ssh server和subversion
```sh
yum install -y openssh-server subversion
```

#### 2. 新建系统用户svnuser，用于ssh登录
```sh
useradd svnuser
```

#### 3. 配置svnuser权限

- 添加开发组 *svngroup*
```sh
groupadd svngroup
chgrp -R svngroup /opt/svn/test
chmod g+w -R /opt/svn/test/db
chmod o=  -R /opt/svn/test/db
ll /opt/svn/test/db
```
- 将 *svnuser* 加入 *svngroup*
```sh
usermod -aG svngroup svnuser
id svnuser
```
- 给 *svnuser* 添加代码库访问权限
```sh
vim /opt/svn/test/conf/authz
```

### 二、linux客户端配置
#### 1. 新建一个用户dev，模拟客户端开发
```sh
su
useradd dev
echo dev | passwd --stdin dev
```

#### 2. 在dev端，创建ssh密钥
```sh
su dev
cd
ssh-keygen -t rsa -b 1024 -C "developer"
#ssh-copy-id svnuser@192.168.244.250
```

#### 3. 在svnuser端，加入dev生成的ssh公钥
- 拷贝 *ssh* 公钥
```sh
su
mkdir /home/svnuser/.ssh
cp -p /home/dev/.ssh/id_rsa.pub /home/svnuser/.ssh/id_rsa_dev.pub
```
- 生成 *authorized_keys*
```sh
cd /home/svnuser/.ssh
echo -n 'command="/usr/bin/svnserve -t -r /opt/svn --listen-port 3690 --tunnel-user=svnuser",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_dev.pub >> authorized_keys
```
- 修改 *ssh* 目录权限
```sh
chmod 700 /home/svnuser/.ssh
chmod 600 /home/svnuser/.ssh/authorized_keys
chown -R svnuser:svnuser /home/svnuser/.ssh
```

#### 4. 在dev端，使用ssh私钥登录svn
- 在用户目录生成 *.subversion*
```sh
su dev
cd
svn co
```
- 拷贝 *ssh* 私钥
```sh
cp -p /home/dev/.ssh/id_rsa /home/dev/.subversion/id_rsa_dev
```
- 配置 *ssh*：在`[tunnels]`内增加一行
```sh
vim /home/dev/.subversion/config
ssh = /usr/bin/ssh -l svn -i /home/dev/.subversion/id_rsa_dev
```

#### 5. 测试ssh免密登录
```sh
svn co svn+ssh://svnuser@192.168.244.250/test/trunk test
cd test
echo "# linux ssh login" >> Changelog.txt
svn add Changelog.txt
svn ci -m "linux ssh login"
```

### 三、windows客户端配置
在windows环境下，svn+ssh方式需要用到三个软件：`puttygen.exe`, `putty.exe`, `TortoiseSVN`
[下载地址](https://pan.baidu.com/disk/main?from=oldversion#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fsvn)

#### 1. 使用WinScp，将ssh私钥拷贝到windows
- 路径：`/home/dev/.subversion/id_rsa_dev`


#### 2. 使用puttygen.exe，将ssh私钥保存为ppk文件
- 点击 *Load*，选中：`id_rsa_dev`
- 点击 *Save private key*，输入文件名：`id_rsa_dev.ppk`

#### 3. 使用putty.exe，配置session
- `Connection`->`Data`：svnuser
- `Connection`->`SSH`->`Auth`：选中id_rsa_dev.ppk
- `Session`->`Host Name`：192.168.244.250
- `Session`->`Saved Sessions`：192.168.244.250，点保存
- 点击Open，输入密码：123
- 输入：exit，退出

#### 4. 使用TortoiseSVN，进行开发
- *TortoiseSVN*->*Settings*->*Network*->*SSH*，配置*SSH Client*：`"C:\Program Files\TortoiseSVN\bin\TortoisePlink.exe" -l svnuser -pw 123`
- *SVN Checkout...*：`svn+ssh://svnuser@192.168.244.250:3690/opt/svn/test/trunk`
