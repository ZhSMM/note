# Linux 常用命令

HTTPS通讯流程：

CSR文件：



```shell
# 查看openssl版本
openssl version
openssl version -a

# 创建PKI密钥

```



证书生成：

##### 创建PKI密钥

指定加密算法：

+ RSA：目前最广泛使用的加密算法
+ DSA：由于安全原因不再推荐使用
+ ECDSA：目前仍未广泛使用
+ EdDSA：目前仍未广泛使用

指定密钥大小：

+ 用于指定密钥大小，密钥越大则越安全，但相对加解密时间也越长
+ 对于RSA加密算法，至少要确保密钥长度为2048位

设置密钥密码：为创建的密钥指定一个密码，来进一步保护密钥的安全性，但同时会带来密钥使用的复杂性

使用openssl创建密钥：

```shell
# 使用genpkey创建密钥，生成的密钥文件中包含了公钥和私钥，密钥文件需要保密
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private.pem

# 使用genrsa创建密钥：该命令专门使用rsa算法生成密钥，较genpkey更方便
openssl genrsa -out private.pem 4096

# 使用pkey查看生成的密钥文件
openssl pkey -in private.pem -text -noout

# 使用pkey提取出公钥，公钥可以任意分发
openssl pkey -in private.pem -pubout -out public.pem

# 使用rsa查看生成的rsa类型的密钥文件
openssl rsa -in private.pem -text -noout -check

# 使用rsa提取公钥文件
openssl rsa -in private.pem -pubout -out public.pem
```

##### 创建csr文件

命令行创建csr文件：

```shell
# 使用req命令创建csr文件（交互模式）
openssl req -new -key private.pem -out cert.csr

# 使用req命令创建csr文件（命令行指定）
openssl req -new -key private.pem -out cert.csr -subj "/C=CN/ST=Hu Nan/L=ChangSha/O=TLS Demo/OU=IT/CN=example.com/emailAddress=admin@qq.com"

# 查看生成的csr文件内容
openssl req -in cert.csr -text -noout -verify
```



基于配置文件创建csr文件：

1、创建配置文件：`vim cert.conf`

``` 
[ req ]
default_bits            = 2048         # 生成的密钥长度，对应参数-newkey rsa:nbits
default_md              = sha256       # 散列算法,对应参数-sha256
default_keyfile         = privkey.pem  # 默认的私钥文件名,对应参数-keyout
prompt                  = no           # 是否提示输入(交互式)

# 使用req命令进行证书签名请求(csr)时，交互提问的内容和默认值。可以将默认是设成从环境变量获取
# 编写格式取决于prompt的值是yes还是no
# man手册'DISTINGUISHED NAME AND ATTRIBUTE SECTION FORMAT'部分
distinguished_name      = req_distinguished_name  

# 使用方法和distinguished_name一样，介绍暂时忽略
# attributes              = req_attributes  

# 该字段定义了一系列要加入到证书中的扩展项(仅针对自签名证书)
# x509_extensions = v3_ca 

# 这个默认是注释的，代表私钥密码
# input_password = secret   # 读取私钥时的密码
# output_password = secret  # 创建私钥时设置的密码

[ req_distinguished_name ]
C  = CN
ST = Hunan
L  = Changsha
O  = TLS Inc.
OU = IT Department
CN = www.example.com
emailAddress = test@qq.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = example.com
DNS.2 = *.example.com
```

2、基于配置文件创建csr文件：

```shell
# 使用-config指定配置文件
openssl req -new -key private.pem -out cert.csr -config cert.conf
```



##### 创建证书

X.509标准：

+ Version：版本号，证书的版本信息
+ Serial Number：序列号，用于唯一标识证书的序列号
+ Issuer：颁发者，标识证书颁发机构（CA）信息
+ Validity：有效期，指定证书的有效起始日期和截止日期
+ Subject：主体，描述证书申请者的信息
+ Subject Public Key Info：包括公钥及其算法
+ Signature：数字签名，由颁发机构使用其私钥生成的数字签名，用于验证证书的完整性和真实性
+ Extention：扩展信息，包含其他可选信息，如密钥用途、基本约束等

创建X.509证书：

```shell
# 使用req命令同时创建私钥和csr文件
# -nodes：表示私钥不设置密码
openssl req -new \
-newkey rsa:2048 \
-keyout cert-key.pem \
-nodes \
-out cert.csr \
-subj "/C=CN/ST=Hu Nan/L=ChangSha/O=TLS Demo/OU=IT/CN=example.com/emailAddress=admin@qq.com"

# 使用x509创建自签名证书（使用自身密钥加密的证书）
openssl x509 \
-req \
-days 365 \
-in cert.csr \
-sha256 \
-signkey cert-key.pem \
-extfile <(printf "subjectAltName=DNS:example.com,DNS:*.example.com") \
-out cert.crt

# 使用req命令创建自签名证书
openssl req \
-x509 \
-newkey rsa:2048 \
-sha256 \
-days 3650 \
-nodes \
-keyout cert-key.pem \
-subj "/C=CN/ST=Hu Nan/L=ChangSha/O=TLS Demo/OU=IT/CN=example.com/emailAddress=admin@qq.com" \
-addext "subjectAltName=DNS:example.com,DNS:*.example.com,IP:127.0.0.1" \
-out cert.crt

# 查看证书文件
openssl x509 -in cert.crt -noout -text
```



证书格式转换：

参数说明：

| 参数     | 说明                           |
| -------- | ------------------------------ |
| -nodes   | 对私钥不加密                   |
| -noout   | 不输出加密的证书内容           |
| -serial  | 输出证书序列号                 |
| -dates   | 显示证书有效期的开始和终止时间 |
| -subject | 输出证书的subject              |



