#Docker化的 Signal Messenger REST API

这个项目围绕 [signal-cli](https://github.com/AsamK/signal-cli) 创建了一个小型的 Docker 化 REST API。

目前，通过 REST 提供以下功能：

- 注册号码
- 使用通过短信收到的验证码验证号码
- 向多个收件人（或群组）发送消息（包括附件）
- 接收消息
- 连接设备
- 创建/列出/删除群组
- 列出/服务/删除附件
- 更新个人资料

以及 [更多功能](https://bbernhard.github.io/signal-cli-rest-api/)


## 入门指南

1. 创建配置目录  
这可以让你通过删除并重新创建容器来更新 `signal-cli-rest-api`，无需重新注册你的 Signal 号码。

```bash
$mkdir-p$HOME/.local/share/signal api
```


2.启动容器

```bash
$ sudo docker run -d --name signal-api --restart=always -p 8080:8080 \
      -v $HOME/.local/share/signal-api:/home/.local/share/signal-cli \
      -e 'MODE=native' bbernhard/signal-cli-rest-api

```

3.注册或链接你的 Signal 号码

在这种情况下，我们将容器注册为次设备，假设你已经将主号码运行并分配到你的手机上。

因此，在浏览器中打开 http://localhost:8080/v1/qrcodelink?device_name=signal-api ，打开手机上的 Signal，进入 设置 > 链接设备 并使用 + 按钮扫描二维码。

4.测试新的REST API

调用 REST API 端点并发送测试消息：将 +4412345 替换为你的 Signal 号码（国际号码格式），将 +44987654 替换为接收者的号码。

```bash
$curl-X POST-H“内容类型：应用程序/json”'http://localhost:8080/v2/send' \
-d'｛“message”：“Test via Signal API！”，“number”：“+4412345”，“receivers”：[“+44987654”]｝'
```

您现在应该已经向“+44987654”发送了一条消息。

##执行模式

`signal-cli-rest-api`支持三种不同的执行模式，可以通过设置`MODE`环境变量来控制。

***`normal`模式：（默认）**为每个REST API请求调用`signal-cli`可执行文件。作为一个Java应用程序，每次REST调用都需要重新启动JVM（Java虚拟机），这会增加延迟，从而导致最慢的操作模式。
***`native`Mode:**每个REST API请求都使用预编译的二进制`signal-cli-native`（使用GraalVM）。这使得每次调用的延迟和内存使用率大大降低。在“armv7”平台上，此模式不可用，并恢复为“正常”模式。由于GraalVM编译器的实验状态，本机模式也可能不太稳定。
*`json-rpc`模式：生成一个基于JVM的`signal-cli`实例作为守护进程。这种模式通常是最快的，但随着JVM的不断运行，需要更多的内存。


|模式|速度|驻留内存使用情况|
|-------------:|:------------|:------------|
|“正常”|：heavy_check_mark：|正常
|“原生”|：heavy_check_mark：：heavy-check_mark：|正常
|`json-rpc` |：heavy_check_mark:：heavy-check-mark:：heavy_check-mark:|已增加


**在“本机”模式下运行“signal-cli-rest”的示例**

```bash
$sudo docker run-d--name signal api--restart=always-p 9922:8080\
-v/home/user/signal-api:/home/.local/share/signal-cli\
-e'MODE=native'bbernhard/signal-cli-rest api
```

这将启动REST服务的一个实例，该实例可通过以下方式访问http://localhost:9922/v2/send.为了保留Signal编号注册，即用于更新，`Signal-cli`配置的存储位置被映射为Docker Volume到本地`/home/user/Signal-api`目录中。


##自动接收时间表

>：警告：此设置仅在正常/本机模式下需要！

[信号cli](https://github.com/AsamK/signal-cli)，此REST API包装器所基于的，建议定期调用“receive”。因此，如果您尚未定期调用“receive”端点，建议在docker-compose.yml文件中设置“AUTO_receive_SCHEDULE”参数。`AUTO_RECEIVE_SCHEDULE`接受cron调度表达式，并在给定时间自动调用`RECEIVE`端点。例如：“0.22***”每天晚上10点接到“电话”。如果你不熟悉cron调度表达式，你可以使用这个[网站](https://crontab.guru).

**警告**呼叫“接收”将从信号服务器获取已注册信号号码的所有消息！因此，如果使用REST API接收消息，那么使用`AUTO_RECEIVE_SCHEDULE`参数不是一个好主意，因为这样可能会丢失一些消息。

##示例

示例`docker compose.yml`文件：

```yaml
版本：“3”
服务：
信号cli-rest api：
图片：bbernhard/signal-cli-rest-api：最新版本
环境：
-MODE=normal#支持的模式：json-rpc、原生、普通
#-AUTO_RECEIVE_SCHEDULE=0 22***#按需启用此参数（见下文描述）
端口：
-“8080:8080”#将docker端口8080映射到主机端口8080。
体积：
-主机系统上的“./signal-cli-config:/home/.local/share/signal-cli”#map“signal-cli-config”文件夹放入docker容器。当注册新号码时，该文件夹包含密码和加密密钥
```

##文档和使用

###API参考

Swagger API文档可在此处找到(https://bbernhard.github.io/signal-cli-rest-api/). 如果您更喜欢基于简单文本文件的API文档，请查看[此处](https://github.com/bbernhard/signal-cli-rest-api/blob/master/doc/EXAMPLES.md).

###博客文章

-[在Azure Web App for Containers中运行Signal Messenger REST API](https://stefanstranger.github.io/2021/06/01/RunningSignalRESTAPIinAppService/)作者：[@stefanstranger](https://github.com/stefanstranger)
-[发送信号消息](https://blog.aawadia.dev/2023/04/24/signal-api/)作者：[@asad awadia](https://github.com/asad-awadia)

###客户端、库和脚本

|名称|类型|语言|描述|维护者|
| ------------- |:------:|:-----:|---|:-----:|
|[pysignalcliestapi](https://pypi.org/project/pysignalclirestapi/)|库| Python |小型Python库|[@bbernhard](https://github.com/bbernhard)
|[信号机器人](https://pypi.org/project/signalbot/)|库| Python |构建Signal机器人的框架|[@filiper](https://github.com/filipre)
|[将cli信号发送到文件](https://github.com/jneidel/signal-cli-to-file)|脚本|JavaScript|将传入的信号消息另存为文件|[@jneidel](https://github.com/jneidel) |

如果您需要更多功能，请**提交工单**或**创建PR**。

##插件

插件机制允许注册自定义端点（具有不同的有效载荷），而无需分叉项目。看看[这里](https://github.com/bbernhard/signal-cli-rest-api/tree/master/plugins)了解详情。

##高级设置
docker容器中可以设置一堆环境变量，以更改一些技术细节。此设置适用于开发人员和高级用户。通常，您不需要在此处更改任何内容-默认值非常好！

*`SIGNAL_CLI_CONFIG_DIR`：指定docker容器内`SIGNAL-CLI`配置目录的路径。默认为`/home/.local/share/signal-cli/`

*`SIGNAL_CLI_UID `：指定docker容器内`SIGNAL-api`用户的UID。默认值为`1000`

*`SIGNAL_CLI_GID`：指定docker容器内`SIGNAL-api`组的GID。默认值为`1000`

*`SWAGER_HOST`：在SWAGGER UI中用于交互式示例的主机（当它在反向代理后面运行时很有用）。默认为SWAGER_IP:PORT。

*`SWAGER_IP`：SWAGGER UI中用于交互式示例的IP。默认为容器ip。

*`SWAGER_USE_HTTPS_AS_PREFERRED_SCHEME`：在SWAGGER UI中使用HTTPS方案作为首选方案。

*`PORT`：默认端口为`8080 `，除非此env变量被设置为相反。

*`DEFAULT_SIGNAL_TEXT_MODE`：允许设置发送消息时应使用的默认文本模式（支持的值：`normal`、`styled`）。该设置仅在“send”方法的有效载荷中没有明确设置“text_mode”的情况下使用。
