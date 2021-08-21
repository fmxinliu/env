# ssh远程管理

## 一、准备工作

### 1) 安装软件包

|      软件包       |              命令               |
| :------------: | :---------------------------: |
|    openssh     |    yum install -y openssh     |
| openssh-server | yum install -y openssh-server |


### 2) 确认安装成功
```sh
rpm -qa | grep ssh
```


### 3) 关闭 GSSAPI 认证（可选）
```
GSSAPIAuthentication no
```

- 服务端 sshd
```sh
vi /etc/ssh/sshd_config
```

- 客户端 ssh
```sh
vi /etc/ssh/ssh_config
```


- 重启ssh服务
```sh
service sshd restart
```

### 4) 新建 2 个系统用户

|  账号   |  密码  |   说明   |
| :---: | :--: | :----: |
| user1 | 111  | ssh客户端 |
| user2 | 222  | ssh服务端 |

```sh
useradd user1
echo 111 | passwd --stdin user1

useradd user2
echo 222 | passwd --stdin user2
```

## 二、Linux配置

### 1) 在ssh客户端创建密钥对
```sh
su - user1
ssh-keygen -t rsa -b 2048 -C "user1@192.168.244.250"
```

### 2) 上传公钥文件到ssh服务端
```sh
scp ~/.ssh/id_rsa.pub user2@192.168.244.250:~
```

### 3) 在ssh服务端导入公钥信息
```sh
ssh user2@192.168.244.250
mkdir ~/.ssh  2>/dev/null
cat id_rsa.pub >> .ssh/authorized_keys
rm -rf id_rsa.pub
```

### 4) 修改ssh授权文件访问权限
```sh
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

### 5) 测试免密登录
```sh
su - user1
ssh  user2@192.168.244.250
```

## 三、Windows配置

[*ssh连接工具下载地址*](https://pan.baidu.com/disk/main?from=oldversion#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fssh)

|    软件包    |    说明    |
| :-------: | :------: |
| SecureCRT |  ssh终端   |
| SecureFX  | sftp传输工具 |

### 1) 拷贝ssh私钥

- 使用 **SecureFX**，将 ssh 私钥 `id_rsa` 拷贝到 windows

### 2) 配置SecureCRT

- **Connection** > **name**：`user2@192.168.244.250`
- **Connection** > **SSH2**
  - **Hostname**：`192.168.244.250`
  - **Port**：`22`
  - **Username**：`user2`
  - 选中`Authentication` > `PublicKey`，点击`Properties...`，添加 `id_rsa`

### 3) 测试免密登录

- 进入**Session Manager**，选中 `user2@192.168.244.250`，双击登录



## 四、sshd安全设定（可选）
```
RSAAuthentication yes     # 开启RSA验证
PubkeyAuthentication yes  # 使用公钥验证

PermitRootLogin no        # 禁止root的ssh登录
PasswordAuthentication no # 禁止密码验证登录
PermitEmptyPasswords no   # 禁止空密码登录
```
