# minikube搭建一个k8s开发环境

#### k8s环境搭建

minikube：[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

minikube搭建k8s环境：

```shell
# 安装minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 查看minikube版本
minikube version

# kubectl是一个与kubernetes、minikube彼此独立的项目，需要独立安装
# 存放目录：.minikube/cache/linux/amd64/v1.28.3/kubectl
# 在minikube环境中，存在两个客户端：minikube管理Kubernetes集群环境，kubectl操作实际的Kubernetes功能
minikube kubectl

# 设置kubectl命令别名
# minikube自带的kubectl有一点形式上的限制，要在前面加上minikube的前缀，后面再有个--
alias kubectl="minikube kubectl --"

# kubectl提供了命令自动补全的功能
source <(kubectl completion bash)

# 启动minikube环境（版本号可选）
minikube start [--kubernetes-version=v1.28.3]

# 关闭minikube环境
minikube stop

# [可选]，安装控制面板
minikube dashboard

# 查看集群状态
minikube status
minikube node list

# 登录到虚拟节点的节点
minikube ssh
```



##### k8s kubectl 报错 c-bash: _get_comp_words_by_ref: 未找到命令

故障现象：

```shell
# kubectl提供了命令自动补全的功能
source <(kubectl completion bash)

# 执行kubectl get时报错
kubectl get pod-bash: _get_comp_words_by_ref: 未找到命令
```

解决问题：

```shell
# 安装bash-completion
yum install bash-completion -y

# 执行bash_completion
source /usr/share/bash-completion/bash_completion

# 重新加载kubectl completion
source <(kubectl completion bash)
```



##### minikube坑点

```shell
# docker需要升级20.10.1以上
# 使用root账号安装时，添加--force
minikube start --force

# 镜像无法拉取
minikube delete
minikube start --image-mirror-country='cn'
```



##### 部署POD

```shell
# 创建POD，使用--image指定镜像
kubectl run ngx --image=nginx:alpine

# 查看POD
kubectl get pod
```



##### 部署Service

```shell
# 创建示例部署并将其公开在端口 8080 上
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080

# 查看部署
kubectl get services hello-minikube

# 访问此服务的最简单方法是让 minikube 为您启动 Web 浏览器
minikube service hello-minikube

# 或者，使用 kubectl 转发端口
# 访问：http://localhost:7080/
kubectl port-forward service/hello-minikube 7080:8080
```



##### 部署LoadBalancer

```shell
# 要访问 LoadBalancer 部署，请使用“minikubetunnel”命令
kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080

# 在另一个窗口中，启动隧道以为“平衡”部署创建可路由 IP
minikube tunnel

# 要查找可路由的 IP，请运行以下命令并检查 EXTERNAL-IP 列
kubectl get services balanced
```



##### 部署Ingress

```shell
# 启用入口插件
minikube addons enable ingress
```

创建简单的echo-server服务和一个用于路由到这些服务的 Ingress 对象：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
    - name: foo-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
    - port: 8080
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
    - name: bar-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
    - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /foo
            backend:
              service:
                name: foo-service
                port:
                  number: 8080
          - pathType: Prefix
            path: /bar
            backend:
              service:
                name: bar-service
                port:
                  number: 8080
---
```

应用内容：

```shell
kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
```



#### 管理集群

minikube管理集群：

```shell
# 暂停 Kubernetes，而不影响已部署的应用程序
minikube pause

# 取消暂停已暂停的实例
minikube unpause

# 停止集群
minikube stop

# 更改默认内存限制（需要重新启动）
minikube config set memory 9001

# 浏览易于安装的 Kubernetes 服务目录
minikube addons list

# 创建运行旧版 Kubernetes 的第二个集群
minikube start -p aged --kubernetes-version=v1.16.1

# 删除所有 minikube 集群
minikube delete --all
```

