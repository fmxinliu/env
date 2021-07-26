### 配置网卡

#### 1. 设置网络
```sh
cp ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0
```

#### 2. DNS服务器
```sh
cat /etc/resolv.conf
```

#### 3. 主机名
```sh
cat /etc/sysconfig/network
cat /etc/hosts
```
#### 4. 重启网络服务
```sh
/etc/init.d/network restart
```

#### 5. 查看网络
```sh
ifconfig
```