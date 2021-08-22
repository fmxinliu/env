# DNS服务

## 一、准备工作

### 1. 安装软件包
```sh
yum -y install bind
```

### 2. 开放防火墙端口
```sh
vi /etc/sysconfig/iptables
```

## 二、配置

|       ip        |  说明  |
| :-------------: | :--: |
| 192.168.244.250 | 主dns |
| 192.168.244.251 | 从dns |

### 1. 主dns

|           配置文件           |   说明   |
| :----------------------: | :----: |
|     /etc/named.conf      | 主配置文件  |
| /etc/named.rfc1912.zones |  区域文件  |
|       /var/named/*       | 解析数据文件 |

```sh
cp -p master/named.conf          /etc/named.conf
cp -p master/named.rfc1912.zones /named.rfc1912.zones
cp -p master/xdl.localhost       /var/named/xdl.localhost
cp -p master/xdl.empty           /etc/named/xdl.empty
```

### 2. 从dns

|           配置文件           |  说明   |
| :----------------------: | :---: |
|     /etc/named.conf      | 主配置文件 |
| /etc/named.rfc1912.zones | 区域文件  |

```sh
cp -p slave/named.conf          /etc/named.conf
cp -p slave/named.rfc1912.zones /named.rfc1912.zones
```

## 三、测试

### 1. 主dns
```sh
# 配置网卡
cp -p test/ifcfg-eth0.master /etc/sysconfig/network-scripts/ifcfg-eth0
service network restart
service named restart

# 测试
$ nslookup
> www.xdl.com
> 192.168.244.250
```

### 2. 从dns
```sh
# 配置网卡
cp -p test/ifcfg-eth0.slave /etc/sysconfig/network-scripts/ifcfg-eth0
service network restart
service named restart

# 测试
$ nslookup
> ftp.xdl.com
> 192.168.244.250
```
