### 部署 SVN+SSH

#### 1. Linux服务器端配置

##### 1) 安装软件包

|     软件包      |             命令              |
| :----------: | :-------------------------: |
|  `openssh`   |  `yum install -y openssh`   |
| `subversion` | `yum install -y subversion` |

##### 2) 新建svn账号

| svn账号  | 访问权限 |
| :----: | :--: |
| `dev1` |  读写  |
| `dev2` |  读写  |

- 编辑权限管理文件

```sh
vim /opt/svn/test/conf/authz
```

- 追加如下配置

```
[groups]
ssh_group = dev1, dev2

[test:/]
@ssh_group = rw
```

##### 3) 新建svn系统账号

|     系统账号     | 登录密码  |      说明       |
| :----------: | :---: | :-----------: |
| `os_svnuser` | `000` | 提供 *svn* 访问服务 |

- 添加系统账号 *os_svnuser*
```sh
useradd os_svnuser
echo 000 | passwd --stdin os_svnuser
```

- 添加权限管理组 *svngroup*
```sh
groupadd svngroup
chgrp -R svngroup /opt/svn/
chmod g+w -R /opt/svn/
chmod o=  -R /opt/svn/
```

- 将 *os_svnuser* 加入 *svngroup*
```sh
usermod -G svngroup os_svnuser
id os_svnuser
```

#### 2. Linux客户端配置 

##### 1) 新建开发账号

|   系统用户    | 登录密码  |      说明      |
| :-------: | :---: | :----------: |
| `os_dev1` | `111` | *linux* 开发账号 |
| `os_dev2` | `222` | *linux* 开发账号 |

```sh
useradd os_dev1
echo 111 | passwd --stdin os_dev1

useradd os_dev2
echo 222 | passwd --stdin os_dev2
```

##### 2)  配置ssh

|     系统用户     |      ssh私钥      |                ssh公钥                 | 映射svn账号 |
| :----------: | :-------------: | :----------------------------------: | :-----: |
| `os_svnuser` |        -        |                  -                   |    -    |
|  `os_dev1`   | `~/.ssh/id_rsa` | `/home/os_svnuser/.ssh/id_rsa_1.pub` | `dev1`  |
|  `os_dev2`   | `~/.ssh/id_rsa` | `/home/os_svnuser/.ssh/id_rsa_2.pub` | `dev2`  |

- 创建 *ssh* 密钥

```sh
su - os_dev1
ssh-keygen -t rsa -b 2048 -C "os_dev1@192.168.244.250"

su - os_dev2
ssh-keygen -t rsa -b 2048 -C "os_dev2@192.168.244.250"
```

- 拷贝 *ssh* 公钥

```sh
su -
mkdir /home/os_svnuser/.ssh
cp -p /home/os_dev1/.ssh/id_rsa.pub /home/os_svnuser/.ssh/id_rsa_1.pub
cp -p /home/os_dev2/.ssh/id_rsa.pub /home/os_svnuser/.ssh/id_rsa_2.pub
```

- 生成 *authorized_keys*

```sh
cd /home/os_svnuser/.ssh

echo -n 'command="/usr/bin/svnserve -t -r /opt/svn --listen-port 3690 --tunnel-user=dev1",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_1.pub >> authorized_keys

echo -n 'command="/usr/bin/svnserve -t -r /opt/svn --listen-port 3690 --tunnel-user=dev2",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_2.pub >> authorized_keys
```

- 修改 *ssh* 目录权限

```sh
chmod 700 /home/os_svnuser/.ssh
chmod 600 /home/os_svnuser/.ssh/authorized_keys
chown -R os_svnuser:os_svnuser /home/os_svnuser/.ssh
```

##### 3) 测试免密登录、开发

```sh
su - os_dev1
svn co svn+ssh://os_svnuser@192.168.244.250/test/trunk test
cd test
echo "- ssh login: os=linux,user_id=os_dev1,svn_id=dev1" >> changelog.txt
svn add changelog.txt
svn ci -m "add changelog.txt"

su - os_dev2
svn co svn+ssh://os_svnuser@192.168.244.250/test/trunk test
cd test
echo "- ssh login: os=linux,user_id=os_dev2,svn_id=dev2" >> changelog.txt
svn ci -m "update changelog.txt"
```