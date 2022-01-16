# [http访问托管库](https://secure.phabricator.com/book/phabricator/article/diffusion_hosting/)

### 1) 新建系统账号

|     系统用户      | 登录密码 |              说明              |
| :-----------: | :--: | :--------------------------: |
|  `www-user`   | 000  |        使用该账号运行*nginx*        |
| `daemon-user` | 000  | 使用该账号运行*phabricator daemons* |

```sh
useradd www-user
echo 000 | passwd --stdin www-user

useradd daemon-user
echo 000 | passwd --stdin daemon-user
```

### 2) 以www-user身份运行nginx

```sh
vi /usr/local/nginx/conf/nginx.conf
# 添加
user www-user;

vi /usr/local/php/etc/php-fpm.d/www.conf
# 添加
user = www-user
```

### 3) 以daemon-user身份运行Phabricator daemons

```sh
cd /usr/local/pha/phabricator/bin/
./config set phd.user daemon-user
```

### 4) 配置www-user以daemon-user身份管理托管库

```sh
vi /etc/sudoers

# 添加
## Allow www-user run commands as daemon-user
www-user ALL=(daemon-user) SETENV: NOPASSWD: /usr/bin/git, /usr/libexec/git-core/git-http-backend

# 注释
#Defaults    requiretty

# 配置文件/目录权限
chmod u+s /usr/local/nginx/sbin/nginx
chmod u+s /usr/local/php/sbin/php-fpm
chmod u+s /var/lock/subsys
```

### 5) 进入Phabricator，配置HTTP

```sh
# 配置>设置： diffusion.allow-http-auth
# 设置>版本控制系统密码： VCS Password
```

### 6) 重启

```sh
reboot
ps aux | grep nginx | grep -v grep   # 查看nginx运行用户
ps aux | grep php-fpm | grep -v grep # 查看php-fpm运行用户
```
