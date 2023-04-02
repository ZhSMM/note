问题列表索引：

1. [markdown实现页面跳转](#markdown实现页面跳转)
   + [页内跳转](#页内跳转)
   + [跨页跳转](#跨页跳转)
   + [GitHub目录/页内跳转](#GitHub目录页内跳转)
2. [git初始化配置](#git初始化配置)
3. [ssh免密登录](#ssh免密登录)



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

