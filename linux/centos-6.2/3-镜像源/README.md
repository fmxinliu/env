- 在线镜像源
```sh
# 备份yum源
cp -p /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak

# 配置yum源
cp CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo

# 清除缓存
yum clean all
rm -rf /var/cache/yum/

# 生成缓存
yum makecache
```

