# docker基本命令

官方网站：[https://docs.docker.com/reference/](https://docs.docker.com/reference/)

#### 镜像操作命令

```shell
# 从远端仓库拉取镜像
docker pull <image_name>:<image_tag>

# 列出当前本地已有的镜像
docker images

# 删除不再使用的镜像
docker rmi <image_name>:<image_tag>

# 镜像导出
docker save ngx-app:latest -o ngx.tar

# 镜像导入
docker load -i ngx.tar
```

注意事项：

1. 镜像IMAGE ID的十六进制字符串是对镜像文件使用SHA256算法的摘要，总长度是64字符（32字节，256位）。
2. 如果一个镜像同时具有多个标签，则不能直接使用IMAGE ID来删除（删除会提示镜像存在多个引用，即标签，拒绝删除）。
3. 在一些特殊情况会导致镜像”丢失“名字或标签，在列表里显示`<none>`



#### 容器操作

```shell
# 从镜像启动容器
# -it：开启交互式操作
# -d：容器在后台运行
# --name：设置容器名称
# --rm：不保存容器，运行完毕自动清除
docker run <image_name>
docker run <image_id>

# 进入容器
docker exec --it <container_id> <command>

# 停止容器
# container_id：可以简写，一般输入前三位即可
docker stop <container_id>

# 再次启动已经停止的容器
docker start <container_id>

# 删除容器
docker rm <container_id>

# 列出正在运行的机器
# -a：查看系统里所有容器，包括已经停止运行的容器
docker ps [-a]
```



##### 文件操作

容器与宿主机文件交互：

1. docker cp，如果源路径是宿主机那么就是把文件拷贝进容器，如果源路径是容器 那么就是把文件拷贝出容器

   ```shell
   # 将文件拷贝到容器
   docker cp <path> <container_id>:<container_path>
   
   # 将文件拷贝到宿主机
   docker cp <container_id>:<container_path> <path>
   ```

2. 目录挂载：`-v/--mount <宿主机路径>:<容器内路径>`

   + -v挂载目录时如果发现源路径不存在时会自动创建
   + -v挂载目录默认是可读可写的，可以加上`:ro`变成只读，可以防止容器意外修改文件

   ```shell
   # 把本机的“/tmp”目录挂载到容器里的“/tmp/”目录
   docker run -d --rm -v /tmp:/tmp:ro nginx:latest
   ```

3. 容器卷：docker volume



##### 网络操作

docker提供的三种网络模式：null、host、bridge

+ null：无网络，允许其他的网络插件来自定义网络连接

+ host：直接使用宿主机网络，所有容器会共享宿主机的IP地址和网卡

  ```shell
  # host模式需要在docker run时使用--net=host参数
  docker run -d --rm --net=host nginx:alpine
  ```

+ bridge：桥接模式，默认网络模式，即通过虚拟网卡连接docker0 bridge，再与宿主机通信。该模式下，可以通过 `-p <本机端口>:<容器端口>` 进行端口挂载

  ```shell
  # 指定网络模式
  docker run -d --rm --net=bridge -p 80:8080 redis:latest
  ```



在宿主机上获取容器的IP地址：

```shell
# 查看容器的IP地址
docker inspect <container_id> | grep IPAddress
```



