# CQHttp Python SDK

本项目为酷 Q 的 CoolQ HTTP API 插件的 Python SDK，封装了 web server 相关的代码，让使用 Python 的开发者能方便地开发插件。仅支持 Python 3.x。

关于 CoolQ HTTP API 插件，见 [richardchien/coolq-http-api](https://github.com/richardchien/coolq-http-api)。

## 用法

首先安装 `cqhttp` 包：

```sh
pip install cqhttp
```

注意可能需要把 `pip` 换成 `pip3`。本 SDK 依赖于 `bottle` 和 `requests` 包，因此它们也会被安装。

也可以 clone 本仓库之后用 `python setup.py install` 来安装。

然后新建 Python 文件，运行 CQHttp 后端：

```py
from cqhttp import CQHttp

bot = CQHttp(api_root='http://127.0.0.1:5700/', token='my-token')


@bot.on_message()
def handle_msg(context):
    bot.send(context, '你好呀，下面一条是你刚刚发的：')
    return {'reply': context['message'], 'at_sender': False}


@bot.on_event('group_increase')
def handle_group_increase(context):
    bot.send(context, message='欢迎新人～', is_raw=True)  # 发送欢迎新人


@bot.on_request('group', 'friend')
def handle_request(context):
    return {'approve': True}  # 同意所有加群、加好友请求


bot.run(host='127.0.0.1', port=8080)
```

### 创建实例

首先创建 `CQHttp` 类的实例，传入 `api_root`，即为酷 Q HTTP API 插件的监听地址，如果你不需要调用 API，也可以不传入。Token 也在这里传入，如果没有配置 token，则不传。

### 事件处理

`on_message`、`on_event`、`on_request` 三个装饰器分别对应三个上报类型，括号中指出要处理的消息类型、事件名、请求类型，一次可指定多个，如果留空，则会处理所有这个上报类型的上报。上例中 `handle_msg` 函数将会在收到任意渠道的消息时被调用，`handle_group_increase` 函数会在群成员增加时调用。

上面三个装饰器装饰的函数，统一接受一个参数，即为上报的数据，具体数据内容见 [事件上报](https://richardchien.github.io/coolq-http-api/#/Post)；返回值可以是一个字典，会被自动作为 JSON 响应返回给 HTTP API 插件，具体见 [上报请求的响应数据格式](https://richardchien.github.io/coolq-http-api/#/Post?id=%E4%B8%8A%E6%8A%A5%E8%AF%B7%E6%B1%82%E7%9A%84%E5%93%8D%E5%BA%94%E6%95%B0%E6%8D%AE%E6%A0%BC%E5%BC%8F)。

### API 调用

在设置了 `api_root` 的情况下，直接在 `CQHttp` 类的实例上就可以调用 API，例如 `bot.send_private_msg(user_id=123456, message='hello')`，这里的 `send_private_msg` 即为 [`/send_private_msg` 发送私聊消息](https://richardchien.github.io/coolq-http-api/#/API?id=send_private_msg-%E5%8F%91%E9%80%81%E7%A7%81%E8%81%8A%E6%B6%88%E6%81%AF) 中的 `/send_private_msg`，API 所需参数直接通过 keyword argument 传入。其它 API 见 [API 描述](https://richardchien.github.io/coolq-http-api/#/API)。

为了简化发送消息的操作，提供了 `send(context, message)` 函数，这里的第一个参数 `context` 也就是上报数据，传入之后函数会自己判断当前需要发送到哪里（哪个好友，或哪个群），无需手动再指定，其它参数仍然可以从 keyword argument 指定，例如 `is_raw=True`。

### 运行实例

使用装饰器定义好处理函数之后，调用 `bot.run()` 即可运行。你需要传入 `host` 和 `port` 参数，来指定服务端需要运行在哪个地址，然后在 HTTP API 插件的配置文件中，在 `post_url` 项中配置此地址。

## 遇到问题

本 SDK 的代码非常简单，如果发现有问题可以参考下源码，可以自行做一些修复，也欢迎提交 pull request 或 issue。
