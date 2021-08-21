###  DHCP服务

|       ip        |   说明    |
| :-------------: | :-----: |
| 192.168.244.250 | dhcp服务器 |
| 192.168.244.251 | dhcp客户端 |

#### 1. 安装dhcp包
```sh
yum -y install dhcp
```

#### 2. 配置dhcp服务器
```sh
cp dhcpd.conf /etc/dhcp/dhcpd.conf
service dhcpd restart
netstat -an | grep :67 ; netstat -an | grep :68  # 查看dhcp端口号
```

#### 3. 配置dhcp客户端
```sh
cp ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0
service network restart
```

#### 4. 查看 ip
```sh
vi /var/lib/dhcpd/dhcpd.leases            # 服务端dhcp租约文件
vi /var/lib/dhclient/dhclient-eth0.leases # 客户端dhcp租约文件
ifconfig
```
