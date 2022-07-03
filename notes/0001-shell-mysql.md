# 通过Shell进行数据库操作

通过Shell可以进行数据库相关操作，如数据导入导出等。

#### MariaDB安装

通过yum命令安装Mariadb：

```sh
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



#### 基本信息

mysql client支持的命令行参数：

- -u：用户名
- -p：用户密码
- -h：服务器的IP地址
- -P：监听端口
- -D：连接的数据库
- -N：不输出列信息
- -B：使用tab代替默认交互分隔符
- -e：执行SQL语句

数据导出相关参数：

- -E：垂直输出
- -H：以HTML格式输出
- -X：以XML格式输出



#### 示例

通过Shell命令行非交互式操作数据库示例：

```sh
#!/bin/sh

# MYSQL连接信息
MYSQL_HOST='127.0.0.1'
MYSQL_PORT=3306
MYSQL_USER='root'
MYSQL_PWD='123456'

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



#### 注意事项

##### 连接数据库

当需要执行多条SQL时，通过上述的方式会存在问题，如每一次都是新的连接，会存在无法选择数据库等，此时可以使用如下方法：

```sh
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" \
-p"${MYSQL_PWD}" <<-EOF 2>/dev/null
    show databases;
EOF
```

上述方式存在暴露密码的问题，要解决这个问题，可以用mysql 数据库的一个特殊配置文件。mysql 数据库使用`$HOME/.my.cnf` 文件来读取特殊的启动命令和设置。其中一项设置是由该用户账户发起的mysql会话的默认密码。要在这个文件中设置默认密码，可以加入下面的内容并设置文件权限：`chmod 400 .my.cnf`

```ini
[client]
password = 123456
```

此时脚本可以不需要输入密码：

```sh
#!/bin/bash
MYSQL=`which mysql`
$MYSQL test -u root <<-EOF
    show databases;
    show tables;
    select * from employees where salary > 4000;
EOF
```



##### 获取执行结果示例

判断执行的返回值：

```sh
#!/bin/sh

# MYSQL连接信息
MYSQL_HOST='127.0.0.1'
MYSQL_PORT=3306
MYSQL_USER='root'
MYSQL_PWD='123456'

# MYSQL数据库信息
DB_NAME='students'
TABLE_NAME='students'

# crate database
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" <<-EOF 2>/dev/null
	CREATE DATABASE ${DB_NAME}
EOF
[ $? -eq 0 ] && echo "created DB" || echo DB already exists

# create table
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" <<-EOF 2>/dev/null
use ${DB_NAME};
CREATE TABLE ${TABLE_NAME}(
    id int,
    name varchar(100),
    mark int,
    dept varchar(4)
);
EOF
[ $? -eq 0 ] && echo "Created table ${TABLE_NAME}" || echo "Table students already exist" 

# delete data
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" <<-EOF 2>/dev/null
    use ${DB_NAME};
	DELETE FROM ${TABLE_NAME};
EOF
```



##### 读取CSV文件

存在如下的csv文件：data

```
1,Navin M,98,CS
2,Kavya N,70,CS
3,Nawaz O,80,CS
4,Hari S,80,EC
5,Alex M,50,EC
6,Neenu J,70,EC
7,Bob A,30,EC
8,Anu M,90,AE
9,Sruthi,89,AE
10,Andrew,89,AE
```

将csv插入到数据库，此时需要逐行读入，并给字符串加上双引号，生成执行语句。解析csv数据并生成执行语句的的方式有如下几种：

- 使用awk，使用-F指定分隔符：

  ```sh
  while read line; do
      query=`echo $line | awk -F, '{printf("%s, \"%s\", %s, \"%s\", $1, $2, $3, $4)}'`
      statement="insert into student values (${query})"
  done < /home/data
  ```

- 使用其他方式：

  ```sh
  while read line; do
      oldIFS=$IFS
      IFS=,
      values=($line)
      values[1]="\"`echo ${values[1]} | tr ' ' '#' `\""
      values[3]="\"`echo ${values[3]}`\""
      query=`echo ${values[@]} | tr ' #' ', '`
      IFS=$oldIFS
      statement=`echo "INSERT INTO $TABLE VALUES($query);"`
      echo "$statement"
  done < /home/data
  ```

完整脚本：

```sh
#!/bin/sh

# MYSQL连接信息
MYSQL_HOST='127.0.0.1'
MYSQL_PORT=3306
MYSQL_USER='root'
MYSQL_PWD='123456'

# MYSQL数据库信息
DB_NAME='students'
TABLE_NAME='students'

# 检查是否存在密码文件
if [ $# -ne 1 ]; then
    echo $0 DATAFILE
    echo
    exit 2
fi

data=$1
while read line;
do
    oldIFS=$IFS
    IFS=,
    # ()：将字符串转换为数组
    values=(${line})
    values[1]="\"`echo ${values[1]} | tr ' ' '#' `\""
    values[3]="\"`echo ${values[3]}`\""
    query=`echo ${values[@]} | tr ' #' ', '`
    IFS=$oldIFS
    statement=`echo "INSERT INTO $TABLE VALUES($query);"`
mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" <<-EOF
    use ${DB_NAME};
    INSERT INTO ${TABLE_NAME} VALUES(${query});
EOF
done < $data

if [[ $? -eq 0 ]]; then
    echo "Wrote data into DB"
fi
```



##### 读取数据库

读取数据库示例：

```sh
#!/bin/sh

# MYSQL连接信息
MYSQL_HOST='127.0.0.1'
MYSQL_PORT=3306
MYSQL_USER='root'
MYSQL_PWD='123456'

# MYSQL数据库信息
DB_NAME='students'
TABLE_NAME='students'

# 用tail去掉表头
depts=`mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" <<-EOF | tail -n +2
    use $DB_NAME;
	SELECT DISTINCT dept FROM $TABLE_NAME;
EOF`

for d in $depts; do
    echo Department: $d
result="`mysql -h"${MYSQL_HOST}" -P"${MYSQL_PORT}" -u"${MYSQL_USER}" << EOF
    use $DB_NAME;
    set @i:=0;
    SELECT @i:=@i+1 as rank, name, mark FROM students WHERE dept="$d" ORDER BY mark DESC;
EOF`"
echo "$result"
done
```





