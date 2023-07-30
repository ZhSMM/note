#### MySQL8相关问题

MySQL 8官方文档：[https://dev.mysql.com/doc/refman/8.1/en/](https://dev.mysql.com/doc/refman/8.1/en/)

##### Public Key Retrieval is not allowed

在连接到mysql8数据库时，报错”Public Key Retrieval is not allowed“：

+ 原因：如果用户使用了 sha256_password 认证，密码在传输过程中必须使用 TLS 协议保护，但是如果 RSA 公钥不可用，可以使用服务器提供的公钥。

+ 解决方式：

  + 指定RSA公钥：可以在连接中通过 ServerRSAPublicKeyFile 指定服务器的 RSA 公钥，或者AllowPublicKeyRetrieval=True参数以允许客户端从服务器获取公钥

    > AllowPublicKeyRetrieval=True可能会导致恶意的代理通过中间人攻击(MITM)获取到明文密码，所以默认是关闭的，必须显式开启

  + 切换加密插件：需要以root用户执行

    ```sql
    # 查看所有用户的加密插件
    SELECT user,plugin FROM mysql.user;
    
    # 修改用户加密插件与密码
    # localhost：仅允许本地登录用户，%代表允许所有IP登录数据库用户
    # plugin：mysql_native_password、caching_sha2_password
    alter user 'root'@'localhost' identified with mysql_native_password by 'pwd';
    
    # 刷新授权表
    flush privileges;
    ```



