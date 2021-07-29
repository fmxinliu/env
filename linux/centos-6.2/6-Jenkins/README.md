###  Jenkins 持续集成工具

[下载地址](https://pan.baidu.com/disk/main?from=oldversion#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2FJenkins)

#### 1. 安装 jdk
```sh
yum -y install jdk-8u301-linux-x64.rpm
```

#### 2. 确认安装成功
```sh
java -version
```

#### 3. 安装 jenkins
```sh
yum -y install jenkins-2.289.2-1.1.noarch.rpm
```

#### 4. 修改端口
```sh
netstat -an | grep :8080         # jenkins默认端口为8080，查看是否被占用
vim /etc/sysconfig/jenkins       # 若被占用，修改端口：JENKINS_PORT="XXXX"
```

#### 5. 启动服务
```sh
service jenkins start
```

#### 6. 查看服务
- pid
```sh
service jenkins status
```
- 端口
```sh
netstat -anp | grep :8080
```

#### 7.配置防火墙，开放端口
- 添加：` -A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT`
```sh
vi /etc/sysconfig/iptables
```
- 重启防火墙
```sh
service iptables restart
```

#### 8.在客户机上，测试端口状态
```
telnet 192.168.244.250 8080
```

#### 9.关闭服务
```sh
service jenkins stop
```

#### 10.配置服务开机自启动
```sh
echo "service jenkins start" >> /etc/rc.d/rc.local
```