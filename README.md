# 告警系统

这是一个基于SpringBoot的告警项目。（毕业设计...目前还在开发和完善中）

该项目的主要目的是为监控系统、程序的异常消息等告警信息提供一个统一的上报接口，方便告警信息的发送。

该项目目前支持通过邮件、微信、QQ三个渠道发送告警信息，你可以通过配置中心选择想要发送的渠道。

该项目主要分为如下四个模块:

- 告警API
- 告警配置中心
- 告警服务端
- 告警发送端

## 告警API(alarm-api)

提供一个统一的告警发送接口，只要把这个接口打包并发布到maven库里，就可以供其他项目引用了。

使用的方式很简单，只要输入告警代号和告警内容就可以了，如下:

``` java

AlarmClient.info(20, "你的程序炸了,快点起来修复.");

```

## 告警配置中心(alarm-web)

提供一个简单的web界面，你可以在这里创建和设置告警的相关信息。一个告警的创建流程大致如下：

- 创建接收人和接收组（已存在则跳过）
- 创建告警信息属于的项目/模块（已存在则跳过）
- 进行告警配置（填写告警名称,选择项目/模块、接收组、发送渠道等）

创建成功后，你就会看见这条告警对应的代号，这个代号就是上述API模块中所说的告警代码。那么接下来就可以通过这个代号来发送告警信息了。

当然，一个代号是可以对应多条告警的，当你通过这个代号发送告警信息的时候，所有属于这个代号的告警都会进行发送。但是，有时候我们只是想发送给这个代号下的某些告警，那么我们就可以通过设置路由来解决这个问题。当我们通过发送一条带路由的告警信息，告警系统会通过路由匹配来确认要发送到哪些告警。
例如，代号233对应有下面这4条告警，接着发送如下告警信息：


|代号|告警名称|接收组|路由|是否发送|
|----|----|----|----|----|
|233|机器CPU监控|运维小组A|monitor.cpu.*|否|
|233|机器硬盘监控|运维小组B|monitor.disk.*|否|
|233|机器内存监控|运维小组C|monitor.ram.*|是|
|233|告警机器监控|告警维护小组|monitor.ram.alarm|是|


``` java

AlarmClient.info(233, "monitor.ram.alarm", "告警机器的内存炸了, 快来看看啊!")

```

如上表，"monitor.ram.alarm"这个路由值能匹配到"monitor.ram.alarm"和"monitor.ram.*"，所以实际上就只有"运维小组C"和"告警维护小组"能接收到告警信息。

注：路由值通过"."来分割词组,其中"\*"代表匹配任意一个词组

(PS:界面不好看或者不好用不要太介意,毕竟没怎么写过前端)

## 告警服务端(alarm-server)

告警API发送的告警信息都是发送至告警服务端处理，告警服务端会根据告警的配置来组织告警文本，然后交给发送端发送。

## 告警发送端(alarm-sender)

负责处理告警服务端递交的告警文本及相关信息，发送给对应的告警接收人。

目前支持邮件、微信、QQ三个渠道。

发送的文本大致如下，你也可以自己重新组织：

```
告警名称:告警系统异常
上报编号:7
项目-模块:告警系统-alarm-server
级别:ERROR
IP:127.0.0.1
时间:2017-03-29 11:55:34
内容:你的程序炸了,快点起来修复.
```

## 使用/运行

先把各个模块中的数据库、邮件、redis等配置修改成你自己的，然后进行打包。

打包后分别运行 java -jar alarm-server.jar 和 java -jar alarm-web.jar，启动服务端和配置中心。

运行 java -jar alarm-server.jar {channel} {name} 启动发送端。例如：

- java -jar alarm-server.jar mail mail_1 (启动邮件发送端)
- java -jar alarm-server.jar qq qq_1 (启动QQ发送端,会要你扫二维码登录QQ)
- java -jar alarm-server.jar wechat wechat_1 (启动微信发送端,会要你扫二维码登录微信)

使用QQ和微信只能把告警信息发送给好友，所以申请个账号作为告警机器人，然后添加上要发送的接收人即可。

(PS:如果有微信企业号，那么就直接使用微信企业号发送吧，机器人容易被封...）