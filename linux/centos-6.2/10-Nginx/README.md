###  Nginx

#### 1. 安装

- 安装前置环境
```sh
yum -y install gcc openssl openssl-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel pcre pcre-devel libxslt libxslt-devel bzip2 bzip2-devel
```

-  [源码包安装](https://pan.baidu.com/disk/main?errmsg=Auth+Login+Sucess&errno=0&from=oldversion&ssnerror=0&#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fnginx)
```sh
# 解压nginx的gz包
tar -zxvf nginx-1.18.0.tar.gz

# 配置
cd nginx-1.18.0
./configure --with-http_stub_status_module --with-http_ssl_module --with-stream

# 编译 && 安装
make && make install

# 安装mysql server
vi ~/.bash_profile
# export PATH=$PATH:/usr/local/nginx/sbin
source ~/.bash_profile
```

#### 2. 配置服务

```sh
# 配置 nginx
vi /usr/local/nginx/conf/nginx.conf

# 检查 nginx.conf
nginx -t

# 启动
nginx

# 查看运行状态
ps aux | grep nginx | grep -v grep

# 开放 80 端口
vi /etc/sysconfig/iptables
# -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
service iptables restart

# 浏览器访问： http://192.168.244.250/
```
