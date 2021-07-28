### 部署 SVN

#### 1. 在线安装 subversion

```sh
yum -y install subversion
```

#### 2. 确认安装成功
```sh
svn --version
```

#### 3. 创建库根目录
```sh
mkdir -p /opt/svn
```

#### 4. 创建测试代码库
```sh
svnadmin create /opt/svn/test
```

#### 5.配置并启动服务：两种方式任选其一
- **单库模式**，访问方式：*svn://xxx.xxx.xxx.xxx:3690*
```sh
svnserve -d -r /opt/svn/test --listen-port 3690
```
-  **多库模式**，访问方式：*svn://xxx.xxx.xxx.xxx:3690/test*
```sh
svnserve -d -r /opt/svn      --listen-port 3690
```

#### 6.查看服务
- pid
```sh
ps -ef | grep svnserve | grep -v grep
```
- 端口
```sh
netstat -anp | grep 3690
```

#### 7.配置防火墙，开放端口
- 添加：` -A INPUT -p tcp -m state --state NEW -m tcp --dport 3690 -j ACCEPT`
```sh
vi /etc/sysconfig/iptables
```
- 重启服务
```sh
service iptables restart
```
#### 8.在客户机上，测试端口状态
```
telnet xxx.xxx.xxx.xxx 3690
```

#### 9.关闭服务
```sh
killall svnserve
```

#### 10.配置服务开机自启动
```sh
echo "/usr/bin/svnserve -d -r /opt/svn --listen-port 3690" >> /etc/rc.d/rc.local
```