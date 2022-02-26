# system对象

system对象是一个全局对象。

### exit

```
function exit(reason)
```

> `@param` `String` `reason` 系统停止的原因

调用此函数来使爬虫立即停止。爬虫停止后，`reason`字符串会作为爬虫停止的原因显示在控制台。一般多用于在爬虫初始化时检查必要的参数或者爬虫运行的必要条件。

注意：多节点情况下，调用此函数的节点会立即停止，但是不会影响其他节点。只有当所有节点都停止之后，爬虫状态才会变成停止。

### sendWebhook

```
function sendWebhook(message)
```

> `@param` `String`或`JS对象` `message` 自定义的消息，JS对象会被自动序列化成JSON字符串发送。

发送一个自定义的Webhook消息。参考[Webhook接口开发](https://web.archive.org/web/20180614213119/http://docs.shenjian.io/develop/platform/webhook/develop.html)