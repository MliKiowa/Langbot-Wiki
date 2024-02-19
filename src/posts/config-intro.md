---
title: QChatGPT的详细配置解读
icon: info
# 这是侧边栏的顺序
order: 3
# 设置作者
author: Lazy
# 设置写作时间
date: 2024-02-13
category:
  - 使用指南
tag:
  - 配置方法
  - 使用指南
---



:::info 目录
[[toc]]
:::

## 配置文件

### 配置文件是什么？

当你下载好本项目后，会发现有一个`templates`文件夹，内部存放的就是即将生成的配置文件的模板

配置文件有以下这些：

- `command.json`：用于存放有关QChatGPT项目命令的
- `pipeline.json`：QChatGPT 3.0新引入的概念，平台适配器获取到消息事件之后，发送给pipeline排队进行处理
- `platform.json`：与消息平台有关的配置
- `plugin-settings.json`：用于配置插件
- `provider.json`：控制AI功能，设置与OpenAI交互的参数，key，模型，反代地址，简单的人格预设等
- `scenario-template.json`：配置人格预设的模板，实现更高级的人格预设
- `sensitive-words.json`：设置敏感词，防止出现不当言论
- `system.json`：设置一些系统信息，如管理员，代理，日志管理，并发数，持续渠道提示信息

以下是对各个配置文件的详细解读

## command.json

设置每个命令的权限配置

普通用户权限级别为 1，管理员（`system.json`中设置的）权限级别为 2。 

在这里设置每个命令的最低权限级别，若设置为1，则用户和管理员均可用，若为2，则仅管理员可用

 需要设置子命令，以点号间隔，如`"plugin.on"` 示例：

```json
{ "plugin": 1, 
  "plugin.on": 2 
}
```

 意思是 plugin 命令的最低权限级别为 1，plugin.on 命令的最低权限级别为 2。 每个命令都有默认的权限级别

若不设置，则使用默认的级别在`pkg/command/operators`里每个命令类有默认的权限级别

## pipeline.json

QChatGPT 3.0新引入的概念，平台适配器获取到消息事件之后，发送给pipeline排队进行处理

可以设置黑白名单、群响应规则、是否检查传入消息、传入消息忽略规则、是否检查敏感词、百度云内容审核配置、传入消息长度设置、限速配置

以下是各项的详细解释

### 黑白名单

```json
"access-control":{
    "mode": "blacklist",
    "blacklist": [],
    "whitelist": []
},
```

`access-control`：黑白名单控制

`mode`：设置模式白名单 `whitelist` 和黑名单 `blacklist` 两种模式

`blacklist`：格式`{type}_{id}`，示例：`"blacklist": [group_12345678，person_12341234],`

`whitelist`：格式`{type}_{id}`，示例：`"whitelist": [group_12345678，person_12341234]`



### 群响应规则

```json
"respond-rules": {
    "default": {
        "at": true,
        "prefix": [
            "/ai", "!ai", "！ai", "ai"
        ],
        "regexp": [],
        "random": 0.0
    }
},
```

`respond-rules`：设置群内响应规则，仅对access-control通过的群生效其中可含有多个key，若要对特定群设置响应规则，则以群号作为键未特指的群，将使用 default 中的响应规则

`at`：`true`或`false`，设置@响应

`prefix`：设置响应前缀

`regexp`：正则匹配

`random`：随机匹配，为1时所有消息都响应

你可以为单个群聊设置特定的响应规则，例如

```json
"respond-rules": {
    "default": {
        "at": true,
        "prefix": [
            "/ai", "!ai", "！ai", "ai"
        ],
        "regexp": [],
        "random": 0.0
    },
    "123456":{
        "at": true,
        "prefix": [
            "/ai", "!ai", "！ai", "ai"
        ],
        "regexp": [],
        "random": 0.0
    },
    "789012":{
        "at": true,
        "prefix": [
            "/ai", "!ai", "！ai", "ai"
        ],
        "regexp": [],
        "random": 0.0
    }
},
```

```json
"income-msg-check": true,
```



### 是否检查传入消息

```json
"income-msg-check": true,
```

`income-msg-check`：是否检查传入的消息

### 传入消息忽略规则

```json
"ignore-rules": {
    "prefix": ["/"],
    "regexp": []
},
```

符合规则的传入消息将被忽略，仅`income-msg-check`为`true`时生效

`prefix`：前缀为`/`的忽略

`regexp`：正则匹配的忽略

### 是否检查敏感词

```json
"check-sensitive-words": true,
```

`check-sensitive-words`：是否检查敏感词，敏感词词库为`sensitive-words.json`里面的

### 百度云内容审核配置

```json
"baidu-cloud-examine": {
    "enable": false,
    "api-key": "",
    "api-secret": ""
},
```

`baidu-cloud-examine`：控制是否使用是否进行百度云内容审核

`enable`：enable=true时启用，一定会检查响应结果，仅`income-msg-check`为true时检查传入消息

`api-key`：百度AI开放平台的`API_KEY`

`api-secret`：百度AI开放平台的`SECRET_KEY`

*`API_KEY`与`SECRET_KEY`是在创建完毕应用后，系统分配给用户的，均为字符串，用于标识用户，为访问做签名验证，可在AI服务控制台中的**应用列表**中查看*

### 传入消息长度设置

```json
"submit-messages-tokens": 3072,
```

`submit-messages-tokens`：传给模型的消息长度限制，不同模型有不同的token数限制，QChatGPT 内部会保存各个模型的（约）最大token数限制

   与此值取最小值，在传给模型API之前进行截断

### 限速配置

```json
"rate-limit": {
    "strategy": "drop",
    "algo": "fixwin",
    "fixwin": {
        "default": 60
    }
}
```

`rate-limit`限速配置， 支持 drop 和 wait 两种策略

`strategy`：`drop`为丢弃，`wait`为等待

`algo`： 使用的算法，目前仅支持 `fixwin` （固定窗口），即一分钟内最多处理多少个请求

`fixwin`：具体速率设置，一分钟内最多处理多少个请求， 支持对特定session指定限速，格式为 {type}_{id}，示例：group_12345678，person_12341234

## platform.json

可以配置：要启用的消息适配器列表，是否跟踪内容函数调用过程，群内回复时是否引用原消息，群内回复时是否at原用户，强制消息延迟范围，长消息处理策略，是否向用户隐藏AI的异常信息

### 消息适配器列表

```json
"platform-adapters": [
    {
        "adapter": "yiri-mirai",
        "enable": false,
        "host": "127.0.0.1",
        "port": 8080,
        "verifyKey": "yirimirai",
        "qq": 123456789
    },
    {
        "adapter": "nakuru",
        "enable": false,
        "host": "127.0.0.1",
        "ws_port": 8080,
        "http_port": 5700,
        "token": ""
    },
    {
        "adapter": "aiocqhttp",
        "enable": false,
        "host": "127.0.0.1",
        "port": 8080
    },
    {
        "adapter": "qq-botpy",
        "enable": false,
        "appid": "",
        "secret": "",
        "intents": [
            "public_guild_messages",
            "direct_message"
        ]
    }
],
```

截至2024年2月13日，目前支持`mirai`、`go-cqhttp`、`aiocqhttp`、`QQ官方机器人`，支持同时开启多个消息适配器，甚至四个全开

```json
{
    "adapter": "yiri-mirai",
    "enable": false,
    "host": "127.0.0.1",
    "port": 8080,
    "verifyKey": "yirimirai",
    "qq": 123456789
},
```

`"adapter": "yiri-mirai"`，无需动，表明是接入`mirai`的配置

`enable`：是否启用，`true`开启

`host`：主机地址，一般不动

`port`：设置端口，默认8080，与`mirai-api-http`里面设置的保持一致

`verifyKey`：密钥，默认yirimirai，与`mirai-api-http`里面设置的保持一致

`qq`：设置机器人QQ号，与当前mirai登录的机器人的QQ号保持一致

```json
{
    "adapter": "nakuru",
    "enable": false,
    "host": "127.0.0.1",
    "ws_port": 8080,
    "http_port": 5700,
    "token": ""
},
```

`"adapter": "nakuru",`，无需动，表明是接入`go-cqhttp`的配置

`enable`：是否启用，`true`开启

`host`：主机地址，一般不动

  `ws_port`：正向WS服务器监听地址，与`go-cqhttp`的`config.yml`里面的保持一致

  `http_port`：http服务器监听地址，与`go-cqhttp`的`config.yml`里面的保持一致

  `token`：验证密钥，，与`go-cqhttp`的`config.yml`里面的保持一致

```json
{
    "adapter": "aiocqhttp",
    "enable": false,
    "host": "127.0.0.1",
    "port": 8080
},
```

`"adapter": "aiocqhttp",`，无需动，表示是接入`shamrock等onebot协议实现的机器人框架`的配置

`enable`：是否启用，`true`开启

`host`：主机地址，一般不动

`port`：设置端口，默认8080，与对应框架里面设置的保持一致

```json
{
    "adapter": "qq-botpy",
    "enable": false,
    "appid": "",
    "secret": "",
    "intents": [
        "public_guild_messages",
        "direct_message"
    ]
}
```

`"adapter": "qq-botpy",`，无需动，表明是接入`QQ官方机器人`的配置

`enable`：是否启用，`true`开启

`appid`：申请到的QQ官方机器人的appid

`secret`：申请到的QQ官方机器人的secret

`intents`：控制监听的事件类型，`public_guild_messages` 频道消息`direct_message` 频道私聊消息  `public_messages` Q群消息（需要权限）

### 是否跟踪内容函数调用过程

```json
"track-function-calls": true,
```

是否跟踪内容函数调用过程，如果开启了，在对话中调用的函数记录也会发给用户

关闭后仅会发给用户最终结果



### 群内回复时是否引用原消息

```json
"quote-origin": false,
```



### 群内回复时是否at原用户

```json
"at-sender": false,
```



### 强制消息延迟范围

```json
"force-delay": [0, 0],
```

强制消息延迟范围，以防风控，单位是秒。

### 长消息处理策略

```json
"long-text-process": {
    "threshold": 256,
    "strategy": "forward",
    "font-path": ""
},
```

`threshold`：阈值，文字长度大于此值的消息会使用长消息处理策略

`strategy`：长消息处理策略，目前支持forward和image

`font-path`： image的渲染字体。未设置时，如果在windows下，会尝试寻找系统的微软雅黑字体，若找不到，则转为forward策略。未设置时，若不是windows系统，则直接转为forward策略。

### 是否向用户隐藏AI的异常信息

```
"hide-exception-info": true
```

 是否向用户隐藏AI的异常信息，如果为true，当请求AI出现异常时，会返回一个错误的提示给用户。而把报错详情输出在控制台。

## plugin-settings.json

这个文件会被复制到`plugins/`目录下

以前插件设置分为 metadata.json switch.json settions.json 

现在统一到这里管理

每次启动时，会同步这里的设置和实际加载的插件设置

**这个文件内容是由程序自动生成的，不要手动添加或删除项目，仅可修改优先级和启用状态**

示例：

```json
"plugins": [
    {
        "name": "Nahida",
        "description": "Hello Nahida",
        "version": "0.1",
        "author": "RockChinQ",
        "source": "",   
        "main_file": "plugins/Nahida/main.py",
        "pkg_path": "plugins/Nahida/",
        "priority": 0,  
        "enabled": true   
    }
]
```

`source`：源码地址，若无内容，则表示这个插件不支持程序自动更新

`priority`： 插件运行优先级，在初始化、事件调用时，高优先级的插件将优先被处理

`enabled`：设置为false时，不会初始化，也不会被调用

## provider.json

配置提供AI聊天接口的信息

```json
"enable-chat": true,
```

`enable-chat`：是否开启AI聊天功能

```json
"openai-config": {
    "api-keys": [
        "sk-1234567890"
    ],
    "base_url": "https://api.openai.com/v1",
    "chat-completions-params": {
        "model": "gpt-3.5-turbo"
    },
    "request-timeout": 120
},
```

`api-keys`：以列表的形式设置若干个api key

`base_url`：设置接口地址，后面一定要加`/v1`

`model`：选择模型

`request-timeout`：设置请求超时时间，以秒为单位，对于耗时较长的模型，建议设置较大值

```json
"prompt-mode": "normal",
"prompt": {
    "default": "如果用户之后想获取帮助，请你说”输入!help获取帮助“。" 
}
```

`prompt-mode`：设置人格预设模式，值为`normal`和`full-scenario`

`normal`时，使用下面的`prompt`，可设置多个

```json
"prompt": {
    "default": "如果用户之后想获取帮助，请你说”输入!help获取帮助“。",
    "help": "如果用户之后想获取帮助，请你说”输入!help获取帮助“。"
}
```

在使用中通过

`!reset <预设名>`指令加载，或使用`!default <预设名>`指令将其设为默认

## scenario-template.json

data/scenario 目录下情景预设的模版

```json
{
    "prompt": [
        {
            "role": "system",
            "content": "You are a helpful assistant. 如果我需要帮助，你要说“输入!help获得帮助”"
        },
        {
            "role": "assistant",
            "content": "好的，我是一个能干的AI助手。 如果你需要帮助，我会说“输入!help获得帮助”"
        }
    ]
}
```

如果你要创建新的人格预设，那么在`data/scenario` 新建一个文件，文件名后边要用，一个情景在一个文件

使用中通过

`!reset <文件名>`指令加载，或使用`!default <文件名>`指令将其设为默认

## sensitive-words.json

没什么好说的，就是敏感词词库

## system.json

设置一下系统信息，如管理员、网络代理、是否上报遥测数据供开发者分析、日志等级、会话消息处理并发数、流水线消息处理并发数、帮助消息

```json
"admin-sessions": [],
```

`admin-sessions`：设置管理员会话，格式为 {type}_{id}，type 为 "group" 或 "person"，如：

```json
"admin-sessions": ["group_123456789", "person_123456789"],
```

```json
"network-proxies": {
    "http": null,
    "https": null
},
```

`network-proxies`：设置网络代理，为正向代理，http和https都要填，例如

```json
"network-proxies": {
    "http": 127.0.0.1:7890,
    "https": 127.0.0.1:7890
},
```

正向代理也可以用环境变量设置：http_proxy 和 https_proxy

```json
"report-usage": true,
```

`report-usage`：是否上报遥测数据供开发者分析，不包含敏感信息

```json
"logging-level": "info",
```

`logging-level`：暂时没用，现在只能通过环境变量 DEBUG=true 来设置调试日志输出

```json
"session-concurrency": {
    "default": 1
},
```

`session-concurrency`：会话消息处理并发数，粒度是每个会话，session格式为 {type}_{id}，未指定的session使用 default 配置

```json
"pipeline-concurrency": 20,
```

`pipeline-concurrency`：流水线消息处理并发数，粒度是整个程序

```json
"help-message": "QChatGPT - 😎高稳定性、🧩支持插件、🌏实时联网的 ChatGPT QQ 机器人🤖\n链接：https://q.rkcn.top"
```

`help-message`：帮助消息，用户发送!help 命令时的输出

## 提出建议

:::tip
欢迎广大用户积极提出意见和建议，帮助我们进一步改善
:::

