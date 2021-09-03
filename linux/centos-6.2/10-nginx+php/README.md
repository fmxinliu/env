###  Nginx

#### 1. 安装

- 安装前置环境
```sh
yum -y install gcc openssl openssl-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel pcre pcre-devel libxslt libxslt-devel bzip2 bzip2-devel
```

- [源码包安装](https://pan.baidu.com/disk/main?errmsg=Auth+Login+Sucess&errno=0&from=oldversion&ssnerror=0&#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fnginx)
```sh
# 解压nginx的gz包
tar -zxvf nginx-1.18.0.tar.gz

# 配置
cd nginx-1.18.0
./configure --with-http_stub_status_module --with-http_ssl_module --with-stream

# 编译 && 安装
make && make install
```

#### 2. 配置服务

```sh
# 配置环境变量
vi ~/.bash_profile
# export PATH=$PATH:/usr/local/nginx/sbin
source ~/.bash_profile

# 配置 nginx
vi /usr/local/nginx/conf/nginx.conf

# 检查 nginx.conf
nginx -t

# 启动 nginx
nginx

# 查看运行状态
ps aux | grep nginx | grep -v grep

# 开放 80 端口
vi /etc/sysconfig/iptables
# -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
service iptables restart

# 浏览器访问： http://192.168.244.250/
```

###  PHP

#### 1. [安装](https://pan.baidu.com/disk/main?errmsg=Auth+Login+Sucess&errno=0&from=oldversion&ssnerror=0&#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fphp)

```sh
# 1.安装openssl-1.0.1
tar -zxvf openssl-1.0.1g.tar.gz
cd openssl-1.0.1g
./config --prefix=/usr/local/openssl-1.0.1
make && make install
/usr/local/openssl-1.0.1/bin/openssl version

# 2.安装gdbm
yum -y install gdbm-devel

# 3.安装libzip-1.8.0
tar -zxvf libzip-1.8.0.tar.gz
cd libzip-1.8.0
mkdir build && cd build/
# cmake-3.21.1-linux-x86_64.sh
chmod 777 cmake-3.21.1-linux-x86_64.sh
./cmake-3.21.1-linux-x86_64.sh
cmake=cmake-3.21.1-linux-x86_64/bin/cmake
$cmake ..
make && make install

# 4.configure: error: off_t undefined; check your library configuration
echo /usr/local/lib64 >> /etc/ld.so.conf
echo /usr/local/lib   >> /etc/ld.so.conf
echo /usr/lib         >> /etc/ld.so.conf
echo /usr/lib64       >> /etc/ld.so.conf
ldconfig -v

# 5.安装php-7.3.2
tar -zxvf php-7.3.2.tar.gz
cd php-7.3.2
./configure --prefix=/usr/local/php \
--with-openssl-dir=/usr/local/openssl-1.0.1 \
--with-curl --with-mysqli --with-pdo-mysql --enable-fpm --enable-mbstring --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib-dir --with-libxml-dir --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curlwrappers --enable-mbregex --with-mcrypt --with-mhash --with-gd --enable-gd-native-ttf --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-ftp --with-bz2 --with-ttf --with-xsl --with-gettext --with-pear --enable-calendar --enable-exif --enable-magic-quotes --with-gdbm
make && make install
```

#### 2. 配置

```sh
# 创建快捷方式
ln -s /usr/local/php/bin/php /usr/local/bin/php

# ERROR: failed to open configuration file '/usr/local/php/etc/php-fpm.conf': No such file or directory (2)
cp -p /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

# ERROR: No pool defined. at least one pool section must be specified in config file
cp -p /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf.conf

# 启动 nginx
/usr/local/php/sbin/php-fpm

# 查看 php-fpm 监听端口
netstat -lntp | grep php-fpm
# www.conf.conf -> listen = 127.0.0.1:9000

# 开放 9000 端口
vi /etc/sysconfig/iptables
# -A INPUT -p tcp -m state --state NEW -m tcp --dport 9000 -j ACCEPT
service iptables restart
```

