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

#### 2. 配置

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

# 设置开机自启
```

#### 3. 开机自启

```sh
cp nginx /etc/init.d/nginx
chmod 755 /etc/init.d/nginx
chkconfig --add nginx           # 添加系统服务,管理方式service nginx ??
chkconfig --level 2345 nginx on # 设置开机启动,启动级别
chkconfig --list nginx          # 查看开机启动配置信息
```

#### 4. 重启

```sh
# 方法1
nginx -s stop
nginx

# 方法2
service nginx restart
```

###  PHP

#### 1. [安装](https://pan.baidu.com/disk/main?errmsg=Auth+Login+Sucess&errno=0&from=oldversion&ssnerror=0&#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fphp)

```sh
# 1.安装openssl-1.0.1
tar -zxvf openssl-1.0.1g.tar.gz
./config --prefix=/usr/local/openssl
make && make install
/usr/local/openssl/bin/openssl version  # 查看版本号

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

# 4.安装libxml2-2.9.8
tar -zxvf libxml.tar.gz
cd libxml2-2.9.8
./configure --prefix=/usr/local/libxml2 --exec-prefix=/usr/local/libxml2 --without-python
make && make install
cd /usr/local/libxml2/bin
xml2-config --version

# 5.configure: error: off_t undefined; check your library configuration
echo /usr/local/lib64 >> /etc/ld.so.conf
echo /usr/local/lib   >> /etc/ld.so.conf
echo /usr/lib         >> /etc/ld.so.conf
echo /usr/lib64       >> /etc/ld.so.conf
ldconfig -v

# 6.安装 autoconf
tar -zxvf autoconf-2.68
cd autoconf-2.68
./configure && make && make install

# 7.安装php-7.3.2
tar -zxvf php-7.3.2.tar.gz
cd php-7.3.2
# (7.1)配置
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-openssl-dir=/usr/local/openssl --with-openssl --with-mysqli --with-curl --with-pdo-mysql --with-ldap --with-gd --enable-fpm --enable-mbstring --enable-pcntl --enable-sockets --enable-opcache --enable-zip
# (7.2)编译安装
make && make install
# (7.3)拷贝配置文件php.ini
cp php.ini-production /usr/local/php/etc/php.ini
# (7.4).安装外部扩展apcu-5.1.16
cd ext/apcu
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
echo "extension=apcu.so" >> /usr/local/php/etc/php.ini
echo "apc.enabled=on"    >> /usr/local/php/etc/php.ini
echo "apc.shm_size=64M"  >> /usr/local/php/etc/php.ini
echo "apc.enable_cli=on" >> /usr/local/php/etc/php.ini
# (7.5)查看安装的扩展
/usr/local/php/bin/php -m
```

#### 2. 配置

```sh
# 创建快捷方式
ln -s /usr/local/php/bin/php /usr/bin/php
 
# ERROR: failed to open configuration file '/usr/local/php/etc/php-fpm.conf': No such file or directory (2)
cp -p /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

# ERROR: No pool defined. at least one pool section must be specified in config file
cp -p /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf

# 启动 php-fpm
/usr/local/php/sbin/php-fpm

# 查看 php-fpm 监听端口
netstat -lntp | grep php-fpm
# www.conf -> listen = 127.0.0.1:9000

# 开放 9000 端口
vi /etc/sysconfig/iptables
# -A INPUT -p tcp -m state --state NEW -m tcp --dport 9000 -j ACCEPT
service iptables restart

# 查看加载的配置文件
php --ini
```

#### 3. 开机自启

```sh
# 设置pid
vi /usr/local/php/etc/php-fpm.conf # ;pid = run/php-fpm.pid 取消注释

# 注册服务
cp php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm
chkconfig --add php-fpm           # 添加系统服务,管理方式service php-fpm ??
chkconfig --level 2345 php-fpm on # 设置开机启动,启动级别
chkconfig --list php-fpm          # 查看开机启动配置信息
```

#### 4. 重启

```sh
# 方法1
ps aux | grep php-fpm | grep -v grep
kill -9 xxx xxx xxx
/usr/local/php/sbin/php-fpm

# 方法2
service php-fpm restart
```

### Phabricator

#### 1. [安装](https://pan.baidu.com/disk/main?errmsg=Auth+Login+Sucess&errno=0&from=oldversion&ssnerror=0&#/index?category=all&path=%2Fpackages%2Flinux%2Fcentos-6.2%2Fphabricator)

```sh
# 1.下载代码
mkdir /usr/local/pha
cd /usr/local/pha
unzip arcanist-master.zip
unzip phabricator-master.zip

# 2.添加可执行权限
chmod 755 phabricator/bin/*
chmod -R 755 phabricator/scripts/*
```

#### 2. 配置

```sh
# 1.初始化mysql数据库
cd phabricator/bin
echo $(cat config)' $@' > config
./config set mysql.host '192.168.244.250:3306'
./config set mysql.port 3306
./config set mysql.user root
./config set mysql.pass 123456
echo $(cat storage)' $@' > storage
./storage upgrade

# 2.配置nginx
cp nginx.conf /usr/local/nginx/conf/nginx.conf

# 3.配置basruri
./config set phabricator.base-uri 'http://192.168.244.250'

# 4.配置environment
./config set environment.append-paths '["/usr/bin","/usr/libexec/git-core"]'

# 5.配置mailers
./config set cluster.mailers '[{"key":"stmp-mailer","type":"smtp","options":{"host":"smtp.163.com","port":994,"user":"??","password":"??","protocol":"ssl"}}]' # 邮箱地址+授权码
./config set metamta.default-address ?? # 邮箱地址

# 6.查看配置项
cat /usr/local/pha/phabricator/conf/local/local.json

# 7.启动php-fpm
service php-fpm restart

# 8.启动nginx
service nginx restart

# 9.启动守护进程
cd /usr/local/pha/phabricator/bin
./phd restart

# 浏览器访问： http://192.168.244.250/
```

#### 3. 开机自启

```sh
cp phd /etc/init.d/phd
chmod 755 /etc/init.d/phd
chkconfig --add phd           # 添加系统服务,管理方式service phd ??
chkconfig --level 2345 phd on # 设置开机启动,启动级别
chkconfig --list phd          # 查看开机启动配置信息
```

#### 4. 重启

```sh
# 方法1
cd /usr/local/pha/phabricator/bin
./phd restart

# 方法2
service phd restart
```
