# CentOS7安装docker

#### 安装

docker安装：[https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)

```shell
# 卸载旧版本
yum remove docker docker-common docker-selinux docker-engine

# 安装yum源管理工具
yum install -y yum-utils device-mapper-persistent-data lvm2

# 设置docker yum源
# 中央仓库
yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo
# 阿里仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 检查可用版本
yum list docker-ce --showduplicates | sort -r

# 安装
yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
yum -y remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# 设置开启自启
systemctl start docker
systemctl enable docker
```



#### 切换镜像源

配置镜像源：

+ 腾讯镜像源：[https://cloud.tencent.com/document/product/1207/45596](https://cloud.tencent.com/document/product/1207/45596)
+ 阿里镜像源：[https://help.aliyun.com/zh/acr/user-guide/accelerate-the-pulls-of-docker-official-images](https://help.aliyun.com/zh/acr/user-guide/accelerate-the-pulls-of-docker-official-images)

1. 编辑配置文件：`vim /etc/docker/daemon.json`

   ```json
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
       "registry-mirrors": [
           "https://euq4hskf.mirror.aliyuncs.com",
           "https://mirror.ccs.tencentyun.com",
           "http://hub-mirror.c.163.com",
           "https://docker.mirrors.ustc.edu.cn",
           "https://registry.docker-cn.com"
       ]
   }
   EOF
   ```

2. 重启docker并确认是否生效：

   ```shell
   # 重启docker
   systemctl daemon-reload
   systemctl restart docker
   
   # 查看配置是否生效
   docker info
   ```

