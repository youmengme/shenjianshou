# page对象

`page`对象在回调函数中传递，表示正在爬去的网页对象，其生命周期对应url的生命周期。一个url被调度开始其生命周期之后，`page`对象随后被创建，并贯穿整个url的生命周期。

## 属性

### url

> ```
> String` `url
> ```

当前网页的链接地址。url的值只能在`beforeDownloadPage`和`afterDownloadPage`函数中修改，在其他回调函数中修改均不会影响后续的回调函数。

### raw

> ```
> String` `raw
> ```

下载的网页原始内容。`beforeDownloadPage`中raw的值为`null`，因为此时还未开始下载。raw的值只能在`afterDownloadPage`回调函数中修改，在其他回调函数中修改均不会影响后续的回调函数。

`page.raw`和`content`的区别：
有的回调函数会回传一个`content`参数，说明也是网页内容，区别在于，`page.raw`是网页的原始下载内容（`afterDownloadPage`函数内部修改后的），`content`基于`page.raw`做了网页链接处理，把其中的相对链接地址替换成了绝对链接地址。

### contextData

> ```
> String` `contextData
> ```

当前网页的附加数据，是`site.addUrl`的时候附加的`contextData`数据。

**注意：**
如果`site.addUrl`的时候，附加的数据是JS对象，此处的`contentData`是JSON字符串，需要`JSON.parse`才能转换成原来的JS对象。

### request

> ```
> JS对象` `request
> ```

当前网页的HTTP请求对象，属性包括`url`、`method`、`data`、`headers`。

- `url` `String`
  等同于`page.url`。

- `method` `String`
  HTTP的请求方式，`GET`或`POST`

- `data` `JS对象`
  POST参数。

- `headers` `JS对象`
  请求的header头。示例值如下：

  ```
  {
    "Cookie": "BAIDUID=8853949B8FD7CEC798A8591AFCD5D42A:FG=1; BDSVRTM=0; BD_HOME=0; BD_LAST_QID=12633460359164860154; BIDUPSID=8853949B8FD7CEC798A8591AFCD5D42A; H_PS_PSSID=25641_1452_21097_17001_22158; PSTM=1517829664; ",
    "User-Agent": "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_10_2) AppleWebKit/536.10 (KHTML, like Gecko) Chrome/63.0.3239 Safari/537.17"
  }
  ```

### response

> ```
> JS对象` `response
> ```

当前网页的HTTP响应对象，属性包括`statusCode`、`statusReason`、`isRedirect`、`redirectLocations`、`headers`。

- `statusCode` `Integer`
  HTTP状态码，发生重定向时，此状态码是重定向后的状态码。参考[HTTP状态码大全](https://web.archive.org/web/20180616054531/http://www.cnblogs.com/lxinxuan/archive/2009/10/22/1588053.html)。

- `statusReason` `String`
  HTTP响应状态字符串，与状态码对应。

- `isRedirect` `Boolean`
  是否发生重定向。

- `redirectLocations` `String数组`
  发生重定向时的重定向地址，发生多次重定向时，数组按重定向的顺序依次记录。

- `headers` `JS对象`
  返回的header头。
  **注意：`Set-Cookie`很可能会有多个，它的值是一个字符串数组**
  示例值如下：

  ```
  {
    "Transfer-Encoding": "chunked",
    "Server": "BWS/1.1",
    "X-Ua-Compatible": "IE=Edge,chrome=1",
    "Connection": "Keep-Alive",
    "Date": "Mon, 05 Feb 2018 11:21:16 GMT",
    "Bduserid": "0",
    "Strict-Transport-Security": "max-age=172800",
    "Cache-Control": "private",
    "Bdpagetype": "1",
    "Cxy_all": "baidu+bb4c4c220a0b63142e909942350c39ec",
    "Set-Cookie": [
      "BDSVRTM=10; path=/",
      "BD_HOME=0; path=/",
      "H_PS_PSSID=1442_21099_18559_17001_22158; path=/; domain=.baidu.com"
    ],
    "Vary": "Accept-Encoding",
    "Bdqid": "0xe5b8b2480000105c",
    "Expires": "Mon, 05 Feb 2018 11:20:22 GMT",
    "Content-Type": "text/html; charset=utf-8",
    "X-Powered-By": "HPHP"
  }
  ```

## 方法

### skip

```
function skip(fieldName)
```

> `@param` `String` `fieldName` 抽取项名，可不传。

此函数可以用来过滤抽取结果。不传参数时，即`page.skip()`，表示丢弃当前整个网页的抽取结果。传`fieldName`时，表示过滤该`field`下的当前抽取结果。

示例1：文章标题中不含”经济学”，过滤掉

```
configs.afterExtractField = function(fieldName, data, page, site) {
  if (fieldName == "article_title") {
    //如果"article_title"的抽取结果中不包含"经济学"，就过滤掉这篇文章
    if (data.indexOf("经济学") == -1) {
      page.skip();
    }
  }
  return data;
};
```



示例2：过滤掉点赞数小于5的评论

```
configs.afterExtractField = function(fieldName, data, page, site) {
  if (fieldName == "comments.agree_count") {
    //如果评论的点赞数小于5，过滤掉"comments"的这条抽取结果
    if (parseInt(data) < 5) {
      page.skip("comments");
    }
  }
  return data;
};
```