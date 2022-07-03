# CentOS替换软件源

#### 版本信息查看

查看版本信息：

```sh
# 查看centos版本
cat /etc/issue
cat /etc/redhat-release
cat /proc/version

# 查看linux内核版本
cat /proc/version
uname -a
uname -r

# 查看系统位数
# 32位的系统中int类型和long类型一般都是4字节，64位的系统中int类型还是4字节的，但是long已变成了8字节
# getconf LONG_BIT | WORD_BIT：分别获取long和word的位数，64位系统中分别为64和32
getconf LONG_BIT
getconf WORD_BIT
file /bin/ls
```



#### 切换为腾讯源

centos换源：

```sh
# 备份系统旧配置文件
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 下载新的CentOS-Base.repo到/etc/yum.repos.d/
version=8
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos"$version"_base.repo

# 更新缓存
yum clean all
yum makecache
```

