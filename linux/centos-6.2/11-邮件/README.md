### 配置邮件

#### 1. 使用sendmail发送邮件
```sh
yum install -y sendmail
yum -y install mailx

# 邮箱服务
vi /etc/php.ini
sendmail_path = /usr/sbin/sendmail -t -i

# 重启服务
service sendmail restart

# 开机自启
chkconfig sendmail on
chkconfig --list | grep sendmail

# 测试局域网邮件（输入内容以.结束）
sendmail os_dev1             # root -> os_dev1
tail /var/spool/mail/os_dev1

sendmail -f os_dev2 os_dev1  # os_dev2 -> os_dev1
tail /var/spool/mail/os_dev1
```

#### 2. 使用外部SMTP服务器来发送邮件
```sh
# 设置邮箱
vi /etc/mail.rc
set from=xxxx@163.com
set smtp=smtp.163.com
set smtp-auth-user=xxxx@163.com # 邮箱名
set smtp-auth-password=         # smtp授权密码
set smtp-auth=login

# 测试外网邮件
echo "This is test mail" | mail -s 'Test Mail' 123456@qq.com
```

