# 部署 SVN+SSH

## 一、Linux服务器端配置

### 1) 安装软件包

|    软件包     |            命令             |
| :--------: | :-----------------------: |
|  openssh   |  yum install -y openssh   |
| subversion | yum install -y subversion |

### 2) 新建demo库
```sh
# 方法1：新建
# mkdir -p /opt/svnrepo
# svnadmin create /opt/svnrepo/demo

# 方法2：解压demo.svn.zip拷贝
cp demo /opt/svnrepo/demo
```

### 3) 新建svn账号

| svn账号  | 访问权限 |
| :----: | :--: |
| `dev1` |  读写  |
| `dev2` |  读写  |
| `dev3` |  读写  |
| `dev4` |  读写  |

- 编辑权限管理文件

```sh
vim /opt/svnrepo/demo/conf/authz
```

- 追加如下配置

```
[groups]
ssh_group = dev1, dev2, dev3, dev4

[demo:/]
@ssh_group = rw
```

### 4) 新建svn系统账号

|     系统账号     | 登录密码 |      说明       |
| :----------: | :--: | :-----------: |
| `os_svnuser` | 000  | 提供 *svn* 访问服务 |

- 添加系统账号 *os_svnuser*

```sh
useradd os_svnuser
echo 000 | passwd --stdin os_svnuser
```

- 添加权限管理组 *svngroup*

```sh
groupadd svngroup
chgrp -R svngroup /opt/svnrepo/
chmod g+w -R /opt/svnrepo/
chmod o=  -R /opt/svnrepo/
```

- 将 *os_svnuser* 加入 *svngroup*

```sh
usermod -a -G svngroup os_svnuser
id os_svnuser
```

## 二、Linux客户端配置

### 1) 新建开发账号

|   系统用户    | 登录密码 |      说明      |
| :-------: | :--: | :----------: |
| `os_dev1` | 111  | *linux* 开发账号 |
| `os_dev2` | 222  | *linux* 开发账号 |

```sh
useradd os_dev1
echo 111 | passwd --stdin os_dev1

useradd os_dev2
echo 222 | passwd --stdin os_dev2
```

### 2)  配置ssh

|     系统用户     |     ssh私钥     |               ssh公钥                | svn账号  |
| :----------: | :-----------: | :--------------------------------: | :----: |
| `os_svnuser` |       -       |                 -                  |   -    |
|  `os_dev1`   | ~/.ssh/id_rsa | /home/os_svnuser/.ssh/id_rsa_1.pub | `dev1` |
|  `os_dev2`   | ~/.ssh/id_rsa | /home/os_svnuser/.ssh/id_rsa_2.pub | `dev2` |

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

# os_dev1 关联 dev1 账号
echo -n 'command="/usr/bin/svnserve -t -r /opt/svnrepo --listen-port 3690 --tunnel-user=dev1",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_1.pub >> authorized_keys

# os_dev2 关联 dev2 账号
echo -n 'command="/usr/bin/svnserve -t -r /opt/svnrepo --listen-port 3690 --tunnel-user=dev2",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_2.pub >> authorized_keys
```

- 修改 *ssh* 目录权限

```sh
chmod 700 /home/os_svnuser/.ssh
chmod 600 /home/os_svnuser/.ssh/authorized_keys
chown -R os_svnuser:os_svnuser /home/os_svnuser/.ssh
```

### 3) 测试免密登录、开发

```sh
# 模拟开发1
su - os_dev1
# 如果是新建的demo库，由于没有创建trunk目录，注意去掉URL最后的trunk
# svn co svn+ssh://os_svnuser@192.168.244.250/demo svntest
svn co svn+ssh://os_svnuser@192.168.244.250/demo/trunk svntest
cd svntest
echo "- ssh login: os=linux,user_id=os_dev1,svn_id=dev1" >> changelog.txt
svn add changelog.txt
svn ci -m "add changelog.txt"

# 模拟开发2
su - os_dev2
svn co svn+ssh://os_svnuser@192.168.244.250/demo/trunk svntest
cd svntest
echo "- ssh login: os=linux,user_id=os_dev2,svn_id=dev2" >> changelog.txt
svn ci -m "update changelog.txt"
```

## 三、Windows客户端配置

### 1) 新建开发账号

|   系统用户    |       说明       |
| :-------: | :------------: |
| `os_dev3` | *windows* 开发账号 |
| `os_dev4` | *windows* 开发账号 |

### 2)  配置ssh

|     系统用户     |      ssh私钥      |               ssh公钥                | 映射svn账号 |
| :----------: | :-------------: | :--------------------------------: | :-----: |
| `os_svnuser` |        -        |                 -                  |    -    |
|  `os_dev3`   | C:\ssh\id_rsa_3 | /home/os_svnuser/.ssh/id_rsa_3.pub | `dev3`  |
|  `os_dev4`   | C:\ssh\id_rsa_4 | /home/os_svnuser/.ssh/id_rsa_4.pub | `dev4`  |

- 创建 ssh 密钥

```sh
su -
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_3 -C "os_dev3@192.168.244.128"
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_4 -C "os_dev4@192.168.244.128"
```

- 拷贝 *ssh* 公钥到 *os_svnuser*

```sh
mkdir /home/os_svnuser/.ssh
cp -p ~/.ssh/id_rsa_3.pub /home/os_svnuser/.ssh/id_rsa_3.pub
cp -p ~/.ssh/id_rsa_4.pub /home/os_svnuser/.ssh/id_rsa_4.pub
```

- 生成 *authorized_keys*

```sh
cd /home/os_svnuser/.ssh

# os_dev3 关联 dev3 账号
echo -n 'command="/usr/bin/svnserve -t -r /opt/svnrepo --listen-port 3690 --tunnel-user=dev3",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_3.pub >> authorized_keys

# os_dev4 关联 dev4 账号
echo -n 'command="/usr/bin/svnserve -t -r /opt/svnrepo --listen-port 3690 --tunnel-user=dev4",' >> authorized_keys
echo -n 'no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ' >> authorized_keys
cat id_rsa_4.pub >> authorized_keys
```

- 修改 *ssh* 目录权限

```sh
chmod 700 /home/os_svnuser/.ssh
chmod 600 /home/os_svnuser/.ssh/authorized_keys
chown -R os_svnuser:os_svnuser /home/os_svnuser/.ssh
```

- 使用 *WinScp*，将 *ssh* 私钥拷贝到 *windows*
  - `/root/.ssh/id_rsa_3` -> `os_dev3` : `C:\ssh\id_rsa_3`
  - `/root/.ssh/id_rsa_4` -> `os_dev4` : `C:\ssh\id_rsa_4`

### 3)  配置TortoiseSVN

| [软件](https://pan.baidu.com/disk/main?from=oldversion#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fsvn) |         说明          |
| :--------------------------------------: | :-----------------: |
|               puttygen.exe               | *转换 ssh* 私钥为 *.ppk* |
|               pageant.exe                |    保存 *.ppk* 到内存    |
|               TortoiseSVN                |    访问 *svn* 代码库     |

这里以 `os_dev3` 为例说明：

- 使用**puttygen.exe**，转换私钥格式
  - 点击 *Load*，选中：`C:\ssh\id_rsa_3`
  - 点击 *Save private key*，输入文件名：`id_rsa_3.ppk`

- 使用**pageant.exe**，添加私钥
  - 点击 *Add Key*，添加：`id_rsa_3.ppk`

- 使用**TortoiseSVN.exe**，测试操作代码库
  - 输入 *URL*：`svn+ssh://os_svnuser@192.168.244.250/demo/trunk`