#### markdown实现页面跳转

##### 页内跳转

markdown 实现页内跳转：

1. 通过HTML标签定义一个锚点，如`<span id="1">锚点</span>`
2. 跳转到锚点位置：
   + 使用markdown的链接语法：`[点击跳转](#1)`
   + 使用HTML的a标签：`<a href="#1">点击跳转</a>`

```xml
<!-- 定义h4标题锚点 -->
<h4 id="1">标题</h4>

<!-- 跳转 -->
<a href="#1">跳转到标题</a>
```

##### 跨页跳转

markdown 实现跨页跳转到锚点：

1. 通过HTML标签在 a.md 文件中定义一个锚点，如`<span id="1">锚点</span>`
2. 在另外一个页面中使用相对路径进行页面跳转，支持markdown的链接语法和HTML的a标签：
   + 使用markdown的链接语法：`[点击跳转](a.md#1)`
   + 使用HTML的a标签：`<a href="a.md#1">点击跳转</a>`

##### GitHub目录/页内跳转

GitHub支持的语法在标准markdown语法的基础上进行了修改，称为Github Flavored Markdow(GFM)。

+ 官方文档：[https://github.github.com/gfm/](https://github.github.com/gfm/)
+ 简单示例：[https://github.com/guodongxiaren/README](https://github.com/guodongxiaren/README)

Github通过锚点（Anchor）来实现页内跳转，其每一个标题就是一个Anchor，当标题中带有字母或空格时，字母统一使用**小写**，空格使用`-`连接，且需要忽略路径分隔符`/`。示例如下：

+ 当标题为`#Hello`，链接写法为`[Hello](#hello)`
+ 当标题为`#Hello World`，链接写法为`[Hello world](#hello-world)`
+ 当标题为`#Hello/World`，链接写法为`[Hello/World](#helloworld)`



#### git初始化配置

git初始化配置：

```sh
# 配置用户名
git config --global user.name "用户名"

# 配置邮箱
git config --global user.email "邮箱地址"

# 查看配置
git config --global --list

# 生成公钥，默认生成在~/.ssh目录
ssh-keygen -t rsa -C '邮箱地址'

# 查看公钥
cat ~/.ssh/id_rsa.pub
```



#### ssh免密登录

ssh免密登录：

```sh
# 查看本机的公钥
cat ~/.ssh/id_rsa.pub

# 在远程服务器的~/.ssh/authorized_keys目录中添加公钥
# 注意authorized_keys的权限必须为600，.ssh文件夹的权限必须为700
chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys
```



### Linux

#### 基础操作

##### CentOS rpm常用命令

rpm：Redhat Package Manager，以数据库记录的方式来管理Linux上的软件包及其依赖。其存在如下缺点：

+ 安装的环境必须与编译时的环境一致或者相当
+ 包与包之间存在着相互依赖的情况
+ 卸载包时需要先把依赖的包卸载掉，如果依赖的包是系统所必须的，那就不能卸载这个包，否则会造成系统崩溃

rpm常用命令：

```shell
# 安装软件包：i（安装）、v（可视化）、h（显示进度条）
# 其他安装参数：--force（强制安装）、--nodeps（忽略依赖安装）
rpm -ivh 包全名

# 升级软件包：U（升级）
rpm -Uvh 包全名

# 卸载软件包：e（卸载）
rpm -e 包全名
rpm -e 包全名 --nodeps # 不检查依赖，直接删除rpm包（建议）

# 查询软件包是否安装：q（查询）
rpm -q 包全名

# 查询当前安装的所有rpm包：a（查询所有）
rpm -qa | grep 包名关键字

# 列出该包中有哪些文件
rpm -ql 包全名

# 列出一个未被安装进系统的RPM包文件中包含有哪些文件
rpm -qilp 包全名

# 列出服务器上的文件属于哪一个rpm包
rpm -qf 文件名

# 解压rpm包：rpm使用cpio打包，因此可以先转换成cpio再解压
rpm2cpio xxx.rpm | cpio -div
```



##### CentOS yum常用命令

yum：基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

yum的配置信息位于`/etc/yum.repos.d`目录下的yum.repos.d文件中，位于同命令下存在许多.repo结尾的文件（即软件源），如CentOS-Base.repo，其配置内容如下：

```ini
[BaseOS]
name=CentOS-$releasever - Base
baseurl=http://mirrors.cloud.aliyuncs.com/$contentdir/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

+ `[BaseOS]`：yum的ID，必须唯一，本地有多个yum源的时候，这里必须是唯一的
+ `name`：yum源名称，`$releasever`变量为当前发行版的大版本号
+ `baseurl`：镜像服务器地址，只能存在一个
+ `mirrorlist`：镜像服务器的地址列表，支持两个参数，`$arch`（cpu体系）、`$base`（cpu的基本体系组）
+ `gpgcheck`：gpg验证，配置为1即开启验证，校验密钥为gpgkey，配置为0即关闭

常用命令：

```shell
# 清空缓存列表
yum clean packages # 清除缓存目录下的软件包，清空的是(/var/cache/yum)下的缓存
yum clean headers  # 清除缓存目录下的 headers
yum clean oldheaders # 清除缓存目录下旧的 headers
yum clean, yum clean all # 清除缓存目录下的软件包及旧的headers

# 显示信息
yum list # 已安装与可安装的软件包
yum list installed # 列出所有已安装的软件包
yum info installed # 列出所有已安裝的软件包信息
yum list/info 软件名 # 显示软件包信息/详细信息
yum deplist 软件名 # 查看程序的依赖情况

# 安装
yum install -y 软件名
yum localinstall 软件名 # 安装本地rpm包

# 删除
yum remove 软件名

# 软件包升级
yum check-update # 检查可更新的程序
yum update # 全部更新，升级所有包，以及升级软件和系统内核，可以更新CentOS的内核到最新版本
yum update package1 # 更新指定程序包package1
yum upgrade package1 # 升级指定程序包package1

# yum组管理
yum grouplist # 查看软件组
yum groupinstall 软件组名 # 安装软件组
yum groupremove 软件组名 # 删除软件组
yum groupinfo 软件组名 # 显示软件组信息
```



### SQL

#### 环境安装

##### CentOS 7通过yum安装MariaDB

通过yum命令安装MariaDB：

```sql
# 在仓库中进行搜索
yum list all | grep mariadb

# 安装mariadb
yum install mariadb mariadb-server.x86_64 

# 查看mariadb服务
systemctl status mariadb

# 启动服务
systemctl start mariadb

# 进入mysql服务
mysql

# 执行相关操作
select version();  # 查看版本
show databases;    # 查看所有数据库
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password'; # 修改密码
\q                 # 退出
```

##### CentOS 7通过yum安装MySQL

在CentOS7中默认安装有MariaDB，但有时需要安装MySQL，且安装完成后可以覆盖掉MariaDB：

```shell
# 通过rpm命令查看
rpm -qa | grep mariadb

# 下载官方的yum repository：https://dev.mysql.com/downloads/repo/yum/
wget https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm

# 安装
yum localinstall mysql80-community-release-el7-7.noarch.rpm

# 更新软件源并进行下载
yum update; yum list | grep mysql

# 安装mysql服务
yum install -y mysql-community-server

# 启动mysql服务
systemctl start mysqld.service
systemctl status mysqld.service

# 从安装日志中找出密码
grep "password" /var/log/mysqld.log

# 进入数据库并重置密码，以及开启远程访问
mysql -uroot -pjd<dD5pkmevq
SQL> ALTER USER 'root'@'localhost' IDENTIFIED with mysql_native_password BY 'pD12345@';
SQL> select host,user,authentication_string,plugin from mysql.user; # 查看用户信息
SQL> update mysql.user set host='%' where user='root';
SQL> flush privileges; 
SQL> exit;
```

注意事项：CentOS 7默认安装的MySQL 8.x版本默认监听ipv6的3306端口，导致程序无法连接

```shell
# 查看监听
> lsof -i:3306
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mysqld  1505 mysql   23u  IPv6  17349      0t0  TCP *:mysql (LISTEN)

# 修改配置文件让mysql监听ipv4
vi /etc/my.cnf

bind-address=0.0.0.0

# 重启mysql服务
systemctl restart mysqld.service
```



#### 基础操作

##### MySQL客户端支持的命令行参数

MySQL Client支持的命令行参数：

| 参数 | 说明                      |
| ---- | ------------------------- |
| -u   | 用户名                    |
| -p   | 用户密码                  |
| -h   | 服务器IP地址              |
| -P   | 监听端口                  |
| -D   | 链接的数据库              |
| -N   | 不输出列信息              |
| -B   | 使用tab代替默认交互分隔符 |
| -e   | 执行SQL语句               |

数据导出相关参数：

| 参数 | 说明           |
| ---- | -------------- |
| -E   | 垂直输出       |
| -H   | 以HTML格式输出 |
| -X   | 以XML格式输出  |

通过shell命令行非交互式操作数据库示例：

```shell
#!/bin/sh

# MYSQL连接信息
MYSQL_HOST='127.0.0.1'
MYSQL_PORT=3306
MYSQL_USER='root'
MYSQL_PWD='123456'

# 定义函数


# MYSQL数据库信息
DB_NAME='test'
TABLE_NAME='test'

# DDL（Data Definition Language）操作
# 删除数据库
drop_db_sql="DROP DATABASE IF EXISTS ${DB_NAME}"
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
      -p"${MYSQL_PWD}" -e "${drop_db_sql}"

# 新建数据库
create_db_sql="create database IF NOT EXISTS ${DB_NAME}"
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
      -p"${MYSQL_PWD}" -e "${create_db_sql}"

# 创建表
drop_table_sql="DROP TABLE IF EXISTS ${DB_NAME}.${TABLE_NAME}"
create_table_sql="create table IF NOT EXISTS ${DB_NAME}.${TABLE_NAME} \
(id int auto_increment primary key, name varchar(20)) ENGINE = InnoDB"
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
      -p"${MYSQL_PWD}" -e "${drop_table_sql}"
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
      -p"${MYSQL_PWD}" -e "${create_table_sql}"
      
# 插入数据
insert_sql="insert into ${DB_NAME}.${TABLE_NAME} values(NULL, 'lisi')"
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
      -p"${MYSQL_PWD}" -e "${insert_sql}"

# 查询
select_sql="select * from ${DB_NAME}.${TABLE_NAME}"
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
      -p"${MYSQL_PWD}" -e "${select_sql}"
```

