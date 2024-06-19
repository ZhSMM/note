# python

#### 设置pip源

设置pip源：

```shell
# 更新pip版本
python -m pip install --upgrade pip
pip3 install --upgrade pip

# 查看当前源
pip3 config list

# 命令行修改源
# 可用的pypi源列表（校园网联合镜像站）：https://mirrors.cernet.edu.cn/list/pypi
# 豆瓣源：https://pypi.douban.com/simple/
# 阿里云：http://mirrors.aliyun.com/pypi/simple/
# 清华大学：https://pypi.tuna.tsinghua.edu.cn/simple/
# 中国科技大学：https://pypi.mirrors.ustc.edu.cn/simple/
pip3 config set global.index-url https://pypi.douban.com/simple/

# 配置多个镜像源的平衡负载（使用空格隔开）
pip config set global.extra-index-url "<url1> <url2>..."

# 手工修改配置文件：
# Linux： ~/.pip/pip.conf
# Windows： %APPDATA%/pip/pip.ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
extra-index-url =
    https://mirrors.aliyun.com/pypi/simple/
    https://mirror.baidu.com/pypi/simple
[install]
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```

