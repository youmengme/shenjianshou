# site对象

`site`对象在回调函数中传递，表示整个爬取站点，其生命周期对应整个爬虫的生命周期。

### addUrl

```
function addUrl(url, options)
```

> `@param` `String` `url` 要添加到待爬队列的链接
>
> `@param` `JS对象` `options` 可选参数对象

此函数用来主动将url添加到待爬队列，平台会自动根据`contentUrlRegexes`正则来判断该url是否应该添加到contentUrl队列。

#### options对象

参数options对象主要用来设置请求参数，可设置属性如下：

- method `String` HTTP请求方式
  可选值`GET`和`POST`，默认`GET`。
- data `JS对象` HTTP的POST数据
  平台默认会对值进行urlencode，不需要主动urlencode。
  POST数据支持文件上传，示例代码如下：

```
//下面这段代码模拟网站 https://tool.lu/base64image/ 的上传文件请求
//下载文件并将文件内容存到变量img中
var img = site.requestUrl("http://mat1.gtimg.com/www/images/qq2012/sogouSearchLogo20140629.png", {
  charset:"ISO-8859-1"
});
//构建上传文件的请求
site.addUrl("https://tool.lu/base64image/upload.html", {
  method: "POST",
  headers: {
    "X-Requested-With": "XMLHttpRequest"
  },
  data: {
    "image": new file(img, "img1.jpg"),//表示将img的内容作为文件上传，文件名为"img1.jpg"
  }
});
```

- contextData `JS对象`或`String` 附加在此url上的数据
  附加的数据可以在`field`定义中直接抽取，也可以在回调函数中通过`page.contextData`访问。如果添加的时候使用的是JS对象，会被自动序列化为JSON字符串，这种情况下，`page.contextData`也是JSON字符串。

- headers `JS对象` HTTP的请求头
  可以单独设置此次请求使用的header。

- reserve `Boolean` 此url是否不判断链接去除直接插入待爬队列
  此值为true时，添加时不进行链接去重的判断。

- noProxy `Boolean` 此请求是否强制不使用代理
  此值为true时，本次请求不会使用代理，为false时，根据爬虫设置决定是否使用代理。一般在爬虫中访问自己的HTTP链接时使用此选项。

- charset `String` 此请求使用的编码
  可以单独设置此次请求的编码方式。对返回的内容直接使用此编码进行解码。在对POST的`data`进行自动编码时，如果`headers`里面设置了`Content-Type`，并且指定了charset，则使用指定的charset，否则使用此charset。

- timeout `Integer` 此请求的超时时间
  可以单独设置此次请求的超时时间，覆盖全局的超时时间。

- base64 `Boolean` 是否对返回内容进行base64编码
  为true时，先对返回内容进行base64编码后再返回，多在直接获取图片内容时使用。

- result `String` 可指定获取更详细的内容
  可设置为”response”，来使`requestUrl`返回一个response对象，而非只返回网页内容。

- dupValue `String` 替换默认的链接去重
  在进行链接去重时，默认使用的是链接本身（POST请求会带上POST的data），如果设置了此值，则直接使用此值进行链接去重。
  使用场景：
  在爬取搜索引擎的搜索结果时，每次搜索结果都是一个临时地址，即使是相同的网页，它的链接地址也在一直变，这样链接去重就失去了意义。这种情况，可以从列表中抽取一些特征值，比如标题、时间等，拼接成字符串，赋值给dupValue，链接去重阶段就会使用这个值进行去重，从而达到去重的目的。

- ignoreCookies `Boolean` 忽略此次请求返回的cookie
  为true时，忽略本次请求返回的cookie，默认每次请求返回的cookie会被自动存储。

- urlEncodeData `Boolean` 是否对POST的data进行urlencode
  默认会进行urlencode，如果请求编码比较特殊，有的键值进行了编码，有的键值不进行编码，需要设置此值为false，然后主动选择编码或不编码。

- enableJS `Boolean` 此次请求是否开启JS渲染
  可单独为此次请求设置是否进行JS渲染，默认由全局的enableJS决定。

- events `JS对象数组` 开启渲染时可以额外执行的模拟操作
  设置JS渲染网页后需要触发的事件。目前只支持点击事件，并且只支持一个，即模拟点击网页上的元素，可以使网页加载新的JS资源，并更新网页内容。事件格式是`{"事件名":xpath}`，xpath是要点击元素的xpath，所以events的值可能时这样的：

  ```
  [
    {
      "click": "//div[@id='more-content']"
    }
  ]
  ```

- retryNum `Integer` 请求失败的重试次数
  默认为0，不进行重试，一般在`requestUrl`的时候使用。

- noFail `Boolean` 此请求处理失败时是否进入失败队列
  默认会进入失败队列，当此值为true时，不进入失败队列。在处理有失效性的链接时，多用此选项。

- noRetry `Boolean` 是否不进行失败重试
  在请求下载失败时，默认会按`retryNum`的次数进行重试。此值为true时，此次请求不进行重试操作，如果是代理导致的下载失败，在切换代理后还会进行重试。

- disableRetry `Boolean` 是否强制不重试
  此值为true时，会进一步禁止代理切换后的重试。

### addScanUrl

```
function addScanUrl(url, options)
```

> `@param` `String` `url` 要添加到scanUrl待爬队列的链接
>
> `@param` `JS对象` `options` 可选参数对象

此函数用来将url添加到scanUrl队列，参数options与`addUrl`完全相同。

### requestUrl

```
function requestUrl(url, options)
```

> `@param` `String` `url` 要请求的链接地址
>
> `@param` `JS对象` `options` 可选参数对象
>
> `@return` `String`或`JS对象` 默认直接返回网页内容。当`options.base64`为true时，返回网页内容的base64编码字符串；当`options.result`为`"response"`时，返回response对象。

此函数用来发送请求，并获取请求的返回内容，可大大增强爬虫获取数据的灵活性。

#### options对象

`options`参数与`addUrl`中的[options对象](https://web.archive.org/web/20180616053831/http://docs.shenjian.io/develop/crawler/doc/objects/site.html#options对象)，除了队列相关的选项不起作用外，其他选项都一致。

#### response对象

- `raw` `String`
  返回的网页内容，当`options.base64`为true时，此内容是base64编码后的内容。

- `statusCode` `Integer`
  HTTP状态码，发生重定向时，此状态码是重定向后的状态码。参考[HTTP状态码大全](https://web.archive.org/web/20180616053831/http://www.cnblogs.com/lxinxuan/archive/2009/10/22/1588053.html)。

- `statusReason` `String`
  HTTP响应状态字符串，与状态码对应。

- `isRedirect` `Boolean`
  是否发生重定向。

- `redirectLocations` `String数组`
  发生重定向时的重定向地址，发生多次重定向时，数组按重定向的顺序依次记录。

- `headers` `JS对象`
  响应的header头。
  **注意：`Set-Cookie`很可能会有多个，它的值是一个字符串数组**
  示例值如下：

  ```
  {
    "Transfer-Encoding": "chunked",
    "BDPAGETYPE": "1",
    "Server": "BWS/1.1",
    "Connection": "Keep-Alive",
    "BDQID": "0xd55cc8d200001c08",
    "P3P": "CP=\" OTI DSP COR IVA OUR IND COM \"",
    "Date": "Mon, 05 Feb 2018 10:17:38 GMT",
    "X-UA-Compatible": "IE=Edge,chrome=1",
    "Cache-Control": "private",
    "Vary": "Accept-Encoding",
    "Set-Cookie": [
      "BDSVRTM=10; path=/",
      "BD_HOME=0; path=/",
      "H_PS_PSSID=1442_21099_18559_17001_22158; path=/; domain=.baidu.com"
    ],
    "Cxy_all": "baidu+e969132750ca3c2b69c1d967e0734955",
    "Expires": "Mon, 05 Feb 2018 10:16:53 GMT",
    "BDUSERID": "0",
    "Content-Type": "text/html; charset=utf-8",
    "X-Powered-By": "HPHP"
  }
  ```

- `request` `JS对象`
  请求的相关信息，记录的信息包括`url`（请求地址）、`method`（请求方式GET/POST）、`headers`（请求头）、`data`（POST参数），示例值如下：

  ```
  {
    "url": "http://www.baidu.com",
    "method": "GET",
    "data": "null",
    "headers": {
      "Cookie": "BAIDUID=DC8921638D5088EEDDC37AAB129B54CC:FG=1; BDSVRTM=0; BD_HOME=0; BIDUPSID=DC8921638D5088EEDDC37AAB129B54CC; H_PS_PSSID=1436_21107_18559_20930; PSTM=1517825858; ",
      "User-Agent": "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; zh-CN) Gecko/20100720 (KHTML, like Gecko) Firefox/56.0.1"
    }
  }
  ```

### requestUrlForLocation

```
function requestUrlForLocation(url, options)
```

> `@param` `String` `url` 要请求的链接地址
>
> `@param` `JS对象` `options` 可选参数对象
>
> `@return` `String` 返回重定向后的链接地址

此函数获取请求重定向后的地址。options中除了`base64`和`result`不生效外，其他选项与`requestUrl`相同。

**注意：**

1. 此函数不会跟进多次重定向，只获取第一次重定向的值
2. 如果未发生重定向，此函数返回`null`

### requestUrlForLocations

```
function requestUrlForLocations(url, options)
```

> `@param` `String` `url` 要请求的链接地址
>
> `@param` `JS对象` `options` 可选参数对象
>
> `@return` `String数组` 返回重定向后的所有链接地址

此函数获取请求的所有重定向地址，数组顺序与重定向的顺序相同。未发生重定向时返回`null`。

### async

```
function async(func, params)
```

> `@param` `function` `func` 需要异步执行的函数
>
> ```
> @param` `任意类型` `params` 此参数会完整地传递给`func
> ```

异步方式执行一段JS代码，多用于添加大量url的场景。示例代码：

```
configs.initCrawl = function(site) {
  site.async(function(params) {
    //site参数需要通过async的第二个参数来传递
    var site = params[0];
    for (var i = 1; i < 100000; i++) {
      site.addUrl("https://shop" + i + ".taobao.com/");
    }
  }, [site]);
};
```



### setCharset

```
function setCharset(charset)
```

> `@param` `String` `charset` 编码格式，一般有`UTF-8`、`GBK`等。

设置解析下载网页内容使用的编码，一般无需设置，平台会自动判断，当发现平台判断不对时，可通过此函数设置，来强制使用某编码。

### setUserAgent

```
function setUserAgent(userAgent)
```

> `@param` `String` `userAgent` User-Agent

设置全局默认UserAgent，爬虫将默认使用此UserAgent。

### addHeader

```
function addHeader(key, value)
```

> `@param` `String` `key` 指定是那个header
>
> `@param` `String` `value` header的值

添加全局默认header，后面的每个请求都会默认加上此header。

### addCookie

```
function addCookie(key, value, domain)
```

> `@param` `String` `key` cookie的键
>
> `@param` `String` `value` cookie的值
>
> `@param` `String` `domain` cookie要加到哪个域名下

添加全局默认cookie，后面的请求默认都会带上此cookie，除非在请求中特殊指明。

### addCookies

```
function addCookies(cookies, domain)
```

> `@param` `String` `cookies` 要添加的cookie键值对
>
> `@param` `String` `domain` 这些cookie要添加到哪个域名下

添加全局默认cookie，`cookies`的格式为”key1=value1; key2=value2”，即跟header中的Cookie格式相同。此方法添加的这些cookie也是全局的，后面的请求默认都会带上这些cookie，除非在请求中特殊指明。

### getCookie

```
function getCookie(key, domain)
```

> `@param` `String` `key` 要获取的cookie的键
>
> `@param` `String` `domain` 要获取哪个域名下的cookie
>
> `@return` `String` 返回对应的cookie的值

获取指定domain下的指定cookie。

### getCookies

```
function getCookies(domain)
```

> `@param` `String` `domain` 要获取哪个domain下的cookie
>
> `@return` `JS对象` 相应domain下所有cookie的键值对

获取指定domain下的所有cookie。

### clearCookies

```
function clearCookies()
```

清空当前的所有cookie。

### changeProxy

```
function changeProxy()
```

主动触发切换代理。

### renderImage

```
function renderImage(html, width, height, format)
```

> `@param` `String` `html` 要渲染成图片的html代码
>
> `@param` `Integer` `width` 网页的宽度，也是图片的宽度
>
> `@param` `Integer` `height` 网页的高度，也是图片的高度
>
> ```
> @param` `String` `format` 图片所用格式，默认`PNG
> ```
>
> `@return` `String` 返回渲染图片的base64编码的字符串

此函数用来将一个网页（或片段）渲染成一张图片。