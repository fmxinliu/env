# 部署 Git+SSH

## 一、Linux服务器端配置

### 1) 安装软件包

| 软件包  |         命令         |
| :--: | :----------------: |
| git  | yum install -y git |

### 2) 新建demo库
```sh
git init --bare /opt/gitrepo/demo.git  # 裸仓库
```

### 3) 新建git系统账号

|     系统账号     | 登录密码 |      说明       |
| :----------: | :--: | :-----------: |
| `os_gituser` | 000  | 提供 *git* 访问服务 |

- 添加系统账号 *os_gituser*

```sh
useradd os_gituser
echo 000 | passwd --stdin os_gituser
```

- 添加权限管理组 *gitgroup*

```sh
groupadd gitgroup
chgrp -R gitgroup /opt/gitrepo/
chmod g+w -R /opt/gitrepo/
chmod o=  -R /opt/gitrepo/
```

- 将 *os_gituser* 加入 *gitgroup*

```sh
usermod -a -G gitgroup os_gituser
id os_gituser
```

## 二、客户端配置

### 1) 新建开发账号

|   系统用户    | 登录密码 |       说明       |
| :-------: | :--: | :------------: |
| `os_dev1` | 111  |  *linux* 开发账号  |
| `os_dev2` | 222  |  *linux* 开发账号  |
| `os_dev3` |  -   | *windows* 开发账号 |
| `os_dev4` |  -   | *windows* 开发账号 |

```sh
useradd os_dev1
echo 111 | passwd --stdin os_dev1

useradd os_dev2
echo 222 | passwd --stdin os_dev2
```

### 2)  配置ssh

|     系统用户     |     ssh私钥     |               ssh公钥                | svn账号  |
| :----------: | :-----------: | :--------------------------------: | :----: |
| `os_gituser` |       -       |                 -                  |   -    |
|  `os_dev1`   | ~/.ssh/id_rsa | /home/os_gituser/.ssh/id_rsa_1.pub | `dev1` |
|  `os_dev2`   | ~/.ssh/id_rsa | /home/os_gituser/.ssh/id_rsa_2.pub | `dev2` |
|  `os_dev3`   | ~/.ssh/id_rsa | /home/os_gituser/.ssh/id_rsa_3.pub | `dev3` |
|  `os_dev4`   | ~/.ssh/id_rsa | /home/os_gituser/.ssh/id_rsa_4.pub | `dev4` |

- 创建 *ssh* 密钥

```sh
su - os_dev1
ssh-keygen -t rsa -b 2048 -C "os_dev1@192.168.244.250"

su - os_dev2
ssh-keygen -t rsa -b 2048 -C "os_dev2@192.168.244.250"

# windows 系统使用 Git Bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_3 -C "os_dev3@192.168.244.128"
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_4 -C "os_dev4@192.168.244.128"
```

- 拷贝 *ssh* 公钥

```sh
su -
mkdir /home/os_gituser/.ssh
cp -p /home/os_dev1/.ssh/id_rsa.pub /home/os_gituser/.ssh/id_rsa_1.pub
cp -p /home/os_dev2/.ssh/id_rsa.pub /home/os_gituser/.ssh/id_rsa_2.pub

# windows 系统使用 WinScp 上传 id_rsa_3.pub、id_rsa_4.pub
```

- 生成 *authorized_keys*

```sh
cd /home/os_gituser/.ssh

# os_dev1
cat id_rsa_1.pub >> authorized_keys

# os_dev2
cat id_rsa_2.pub >> authorized_keys

# os_dev3
cat id_rsa_3.pub >> authorized_keys

# os_dev4
cat id_rsa_4.pub >> authorized_keys
```

- 修改 *ssh* 目录权限

```sh
chmod 700 /home/os_gituser/.ssh
chmod 600 /home/os_gituser/.ssh/authorized_keys
chown -R os_gituser:os_gituser /home/os_gituser/.ssh
```

### 3) 测试免密登录、开发

```sh
# dev1提交
su - os_dev1
git config --global user.name "os_dev1"
git config --global user.email "os_dev1@192.168.244.250"
git clone git+ssh://os_gituser@192.168.244.250/opt/gitrepo/demo.git gittest
cd gittest
echo "- ssh login: os=linux,user_id=os_dev1" >> changelog.txt
git add changelog.txt
git commit -m "add changelog.txt"
git push origin master

# dev2提交
su - os_dev2
git config --global user.name "os_dev2"
git config --global user.email "os_dev2@192.168.244.250"
git clone git+ssh://os_gituser@192.168.244.250/opt/gitrepo/demo.git gittest
cd gittest
echo "- ssh login: os=linux,user_id=os_dev2" >> changelog.txt
git add changelog.txt
git commit -m "update changelog.txt"
git push origin master

# dev3提交
git config --global user.name "os_dev3"
git config --global user.email "os_dev3@192.168.244.128"
git clone git+ssh://os_gituser@192.168.244.250/opt/gitrepo/demo.git gittest
cd gittest
echo "- ssh login: os=windows,user_id=os_dev3" >> changelog.txt
git add changelog.txt
git commit -m "update changelog.txt"
git push origin master

# dev4提交
git config --global user.name "os_dev4"
git config --global user.email "os_dev4@192.168.244.128"
git clone git+ssh://os_gituser@192.168.244.250/opt/gitrepo/demo.git gittest
cd gittest
echo "- ssh login: os=windows,user_id=os_dev4" >> changelog.txt
git add changelog.txt
git commit -m "update changelog.txt"
git push origin master

# 查看提交记录
git log origin/master
```