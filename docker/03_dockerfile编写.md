# dockerfile编写

镜像内部有许多的镜像层“Layer”组成，每层都是只读不可修改的一组文件，相同的层可以在镜像之间共享，然后多个层堆叠，再使用一种的”Union FS 联合文件系统“的技术合并在一起，就形成了容器最终的文件系统。

```shell
# 查看镜像的分层信息：分层信息在RootFS部分
docker inspect <image_name>:<image-tag>
```

Dockerfile是一个纯文本，里面记录了一系列的构建指令，如选择基础镜像、拷贝文件、运行脚本等，每个指令都会生成一个Layer，而Docker顺序执行这个文件里的所有步骤，最后就会创建出一个新的镜像出来。



RUN和ARG：

+ ARG创建的变量只在镜像构建过程中可见，容器运行时不可见
+ ENV创建的变量不仅能够在构建镜像的过程中使用，在容器运行中也能够以环境变量的形式被应用程序使用



镜像标签一般格式：

+ 应用版本号（主版本号 + 次版本号 + 补丁号）加上操作系统，示例：nginx:1.21.6-alpine
+ 有的标签还会加上slim、fat，表示这个镜像是经过精简的，还是包含了较多的辅助工具



#### 简单示例

简单的Dockerfile实例：

```dockerfile
# 选择基础镜像
FROM busybox:latest

# 启动容器时默认运行的命令
CMD echo "Hello world"
```

创建镜像：

```shell
# -f：指定Dockerfile文件名
# -t：指定镜像的标签（tag）
# .：build context，构建上下文
docker build -f Dockerfile .
```



#### 最佳实践

##### 书写原则

Dockerfile书写原则：

1. 单一职责：不同功能的应用应该尽量拆分为不同的容器，每个容器只负责单一业务进程

2. 提供注释信息

3. 保持容器最小化：应该避免安装无用的软件包

4. 合理选择基础镜像：容器的核心时用用，只要基础镜像能够满足应用的运行环境即可

   + 关注镜像的安全和大小的时候，一般选择Alpine、BusyBox
   + 关注应用的运行稳定性，一般选择Ubuntu、Debian、CentOS

5. 使用 .dockerignore 文件：

   | 规则         | 含义                                                         |
   | ------------ | ------------------------------------------------------------ |
   | `#`          | # 开头的表示注释，# 后面的所有内容将会忽略                   |
   | `*/tmp*`     | 匹配当前目录下任何以tmp开头的文件或文件夹                    |
   | `*.md`       | 匹配以 .md 为后缀的任意文件                                  |
   | `tem?`       | 匹配以 tem 开头并且以任意字符结尾的文件，? 代表任意一个字符  |
   | `!README.md` | ! 表示排除忽略。<br />例如 .dockerignore 定义如下：<br /><br />*.md<br />!README.md<br /><br />表示除了README.md文件外所有以 .md 结尾的文件。 |

6. 尽量使用构建缓存：

   + 从当前构建层开始，比较所有的子镜像，检查所有的构建指令是否与当前完全一致，如果不一致，则不适用缓存
   + 一般情况下，只需要比较构建指令即可判断是否需要使用缓存，但有些指令除外（例如ADD和COPY）
   + 对于ADD和COPY指令要校验命令是否一致，还要为即将拷贝到容器的文件计算校验和，命令和校验和完全一致，才认为命中缓存

   示例：

   ```dockerfile
   FROM centos:7
   
   # 设置环境变量指令放前面
   ENV PATH /usr/local/bin:$PATH
   # 安装软件指令放前面
   RUN yum install -y make
   # 将业务软件的配置、版本等经常变动的步骤放最后
   ```

7. 正确设置时区：

   ```dockerfile
   # Ubuntu和Debian系统
   RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
   RUN echo "Asia/Shanghai" >> /etc/timezone
   
   # CentOS系统
   RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
   ```

8. 使用国内软件源加快镜像构建速度

9. 最小化镜像层数





##### RUN

RUN指令在构建时将会生成一个新的镜像层并且执行RUN指令后面的内容：

+ 当RUN指令后面跟内容比较复杂时，建议使用反斜杠（\）结尾并且换行
+ RUN指令后面的内容尽量按照字母顺序排序，提高可读性

```dockerfile
FROM centos:7
RUN yum install -y automake \
    curl \
    python \
    vim
```



##### CMD 与 ENTRYPOINT

CMD 与 ENTRYPOINT 相同点：

```dockerfile
# 支持的语法格式
CMD/ENTRYPOINT["command", "param"]
CMD/ENTRYPOINT command param
```

不同点：

+ Dockerfile 中如何使用了 ENTRYPOINT 指令，启动 Docker 容器时需要使用 --entrypoint 参数才能覆盖 Dockerfile 中的 ENTRYPOINT 指令，而使用 CMD 设置的命令则可以被 docker run 后面的参数直接覆盖
+ ENTRYPOINT 指令可以结合 CMD 指令使用，也可以单独使用，而 CMD 指令只能单独使用

选择策略：尽量使用exec模式

+ 如果希望镜像足够灵活，推荐使用 CMD 指令

+ 如果镜像只执行单一的具体程序，并且不希望用户在docker run时覆盖默认程序，建议使用 ENTRYPOINT

+ 如果需要镜像在启动容器时执行一些初始化操作，此时可以使用ENTRYPOINT执行命令，CMD传递参数

  ```dockerfile
  FROM centos:7
  
  ENTRYPOINT ['sh', 'startup.sh']
  CMD['init']
  ```

  启动脚本示例：

  ```shell
  #!/usr/bin/env bash
  
  main() {
    if [ "$1" = 'init' ]; then
        # 初始化操作
    fi
    
    exec "$@"
  }
  
  main "$@"
  ```



##### ADD与COPY

ADD与COPY：

+ COPY：只支持基本的文件和文件夹拷贝功能
+ ADD：支持更多文件按来源类型，比如自动提取 tar 包，并且可以支持源文件为 URL 格式

使用建议：

+ 推荐使用COPY，因为行为更加清晰
+ 仅当将本地压缩文件拷贝到镜像内部时使用ADD指令



##### WORKDIR

推荐使用WORKDIR指定容器的工作路径



