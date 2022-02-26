# console对象

console对象是一个全局对象，主要提供日志打印函数。

神箭手平台提供四个日志级别，从低到高依次为`debug`、`info`、`warn`、`error`。`info`及以上级别的日志会打印到用户的运行日志里面，`debug`只在测试的时候打印。

**日志打印太多会影响爬虫的运行速度，建议合理调整日志级别。**

### log

```
function log(message)
```

> `@param` `String` `message` 要打印的消息

此函数等同于`info`函数，打印info级别的日志。

### debug

```
function debug(message)
```

> `@param` `String` `message` 要打印的消息

打印debug级别的日志。

### info

```
function info(message)
```

> `@param` `String` `message` 要打印的消息

打印info级别的日志。

### warn

```
function warn(message)
```

> `@param` `String` `message` 要打印的消息

打印warn级别的日志。

### error

```
function error(message)
```

> `@param` `String` `message` 要打印的消息

打印error级别的日志。