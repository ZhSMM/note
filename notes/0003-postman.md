# Postman

Postman 下载：[https://www.postman.com/downloads/](https://www.postman.com/downloads/)

Postman 文档：[https://postman.org.cn/sending-requests/variables/#variables-quick-start](https://postman.org.cn/sending-requests/variables/#variables-quick-start)

##### Postman请求

在Postman中构建请求时可以使用URL字段和“参数”选项卡随请求发送路径和查询参数：

- 查询参数附加到请求 URL 的末尾，`?`并以键值对的形式列出，并`&`使用以下语法分隔：`?id=1&type=new`
- 路径参数构成请求 URL 的一部分，并使用前面的占位符引用`:`，如下例所示：`/customer/:id`

![路径参数](https://cdn.jsdelivr.net/gh/ZhSMM/pictures//imgpath-param-v9.jpg)



##### Postman变量

从最宽到最窄的顺序，这些范围是：global、collection、environment、data和local。

- 全局变量使您能够访问集合、请求、测试脚本和环境之间的数据。全局变量在整个工作区中都可用。由于全局变量在 Postman 中具有最广泛的可用范围，因此它们非常适合测试和原型设计，但在以后的开发阶段应使用更具体的范围。
- 集合变量在集合中的整个请求中都可用，并且独立于环境。集合变量不会根据所选环境而改变。如果您使用单一环境，例如用于身份验证或 URL 详细信息，则集合变量是合适的。
- 环境变量使您能够将工作范围限定到不同的环境，例如本地开发与测试或生产。一次可以激活一个环境。如果你有一个单一的环境，使用集合变量会更有效，但是环境允许你指定基于角色的访问级别。
- 数据变量来自外部 CSV 和 JSON 文件，用于定义在使用Newman或Collection Runner运行集合时可以使用的数据集。数据变量具有当前值，在请求或收集运行之后不会持续存在。
- 局部变量是在请求脚本中访问的临时变量。局部变量值的范围仅限于单个请求或收集运行，并且在运行完成后不再可用。如果您需要一个值来覆盖所有其他变量范围但不希望该值在执行结束后持续存在，则局部变量是合适的。

注意：

- 如果在两个不同的作用域中声明了同名变量，则将使用存储在具有最窄作用域的变量中的值。例如，如果有一个名为 的全局变量`username`和一个名为`username`的局部变量，则请求运行时将使用局部值。
- Postman将变量存储为字符串，因此需要在存储对象或数组时使用`JSON.stringify()`转换成字符串，在使用时使用`JSON.parse()`转换成对象。

在脚本中定义变量：

| 方法                     | 用例                                   | 例子                                                         |
| :----------------------- | :------------------------------------- | :----------------------------------------------------------- |
| `pm.globals`             | 用于定义全局变量                       | `pm.globals.set("variable_key", "variable_value");`          |
| `pm.collectionVariables` | 用于定义集合变量                       | `pm.collectionVariables.set("variable_key", "variable_value");` |
| `pm.environment`         | 用于在当前选择的环境中定义一个环境变量 | `pm.environment.set("variable_key", "variable_value");`      |
| `pm.variables`           | 用于定义局部变量                       | `pm.variables.set("variable_key", "variable_value");`        |
| `unset`                  | 您可以使用`unset`删除变量              | `pm.environment.unset("variable_key");`                      |

使用变量：`{{paramName}}`

动态变量：

- `{{$guid}}`: v4 风格的 GUID
- `{{$timestamp}}`: 当前的 Unix 时间戳，以秒为单位
- `{{$randomInt}}`: 0 到 1000 之间的随机整数



##### 使用Pre-request Script实现自动获取Token并写入请求头

可以通过Pre-request Script实现自动获取Token并写入请求头：

```js
const lastLogin = pm.collectionVariables.get("lastLogin");
const current = parseInt(+new Date().getTime() / 1000);
console.log(lastLogin, current);
if (typeof lastLogin == "undefined" || lastLogin == null || lastLogin == "" || current - parseInt(lastLogin) > 60 * 15) {
    const loginUrl = pm.collectionVariables.get("url");
    const username = pm.collectionVariables.get("username");
    const password = pm.collectionVariables.get("password");
    const LoginReq = {
        url: LoginUrl,
        method: 'POST',
        header: {
            'Content-Type': 'application/json;charset=UTF-8',
            'Accept': 'application/json;charset=UTF-8'
        },
        body: {
            mode: 'raw',
            raw: JSON.stringify({
                username: username,
                password: password 
            })
        }
    };

    pm.sendRequest(LoginReq, function (err, response) {
        pm.request.headers.add({
            key: 'Token',
            value: response.json().result.token
        });
        pm.request.headers.add({
            key: 'Content-Type',
            value: 'application/json;charset=UTF-8'
        });
    });
    
    pm.collectionVariables.set("lastLogin", "" + current);
} else {
    pm.request.headers.add({
        key: 'Token',
        value: response.json().result.token
    });
    pm.request.headers.add({
        key: 'Content-Type',
        value: 'application/json;charset=UTF-8'
    });
}
```



##### Postman控制台

在调试Postman的脚本时，可能需要使用到控制台，有两种打开方式：

- 菜单：View -> Show Postman Console

  ![image-20220704223318828](https://cdn.jsdelivr.net/gh/ZhSMM/pictures//imgimage-20220704223318828.png)

- 左下角图标：

  ![image-20220704223405621](https://cdn.jsdelivr.net/gh/ZhSMM/pictures//imgimage-20220704223405621.png)



##### Postman第三方库

参考：[https://blog.csdn.net/fantian_/article/details/106519479](https://blog.csdn.net/fantian_/article/details/106519479)

```js
// 生成MD5字符串
var hashed  = CryptoJS.MD5("待加密的字符串").toString();

// 文件转base64字符串
s = 'Hello';
utf8 = CryptoJS.enc.Utf8.parse(s); // 先转化成UTF-8编码的字符串
b64 = CryptoJS.enc.Base64.stringify(utf8); // 用CryptoJS第三方库(Postman已经内置了)进行编码
```

