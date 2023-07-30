#### Windows安装MySQL

在Windows上安装MySQL：

1. 下载：[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)

2. 在解压路径创建配置文件：my.ini

   ```ini
   [mysqld]
   # 设置3306端口
   port=3306
   # 设置mysql的安装目录
   basedir=D:\install\mysql-8.0.23-winx64
   # 设置mysql数据库的数据的存放目录
   datadir=D:\install\mysql-8.0.23-winx64\data
   # 允许最大连接数
   max_connections=200
   # 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
   max_connect_errors=10
   # 服务端使用的字符集默认为UTFMB4
   character-set-server=utf8mb4
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   # 默认使用“mysql_native_password”插件认证
   default_authentication_plugin=mysql_native_password
   
   [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8mb4
   
   [client]
   # 设置mysql客户端连接服务端时默认使用的端口
   port=3306
   default-character-set=utf8mb4
   ```

3. 将解压路径的bin目录配置到环境变量Path中

4. 以管理员身份运行cmd，执行如下命令进行安装：

   ```shell
   # 初始化：在控制台输出初始化的root密码
   mysqld --no-defaults --initialize --console
   
   # 安装：如果提示服务存在，使用sc delete mysql
   mysqld --install mysql
   
   # 启动服务
   net start mysql
   ```

5. 修改root密码：

   ```shell
   # 进入数据库shell
   mysql -uroot -p
   
   # 修改密码
   alter user 'root'@localhost identified by '1234';
   ```