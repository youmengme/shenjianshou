# field对象

field对象用来定义爬取结果的一个字段，通常我们也叫一个抽取项。下面逐一介绍该对象的属性。

### name

**String类型** **不可为空**

> 抽取项的名字。

**注意：**

1. 名字中不能包含英文句点，即`.`。
2. 同一层级名字不能重复。
3. 为了更好的兼容性（尤其是数据库发布），名字最好不要使用中文，命名尽量符合变量名（标识符）的命名规则。
4. 此值将作为抽取项的标识，如果中途做了修改，将导致之前的数据无法读出该字段。

### alias

**String类型**

> 抽取项的别名。一般起中文名，方便查看数据。只影响网页的上显示，可随意修改。

### selectorType

**枚举类型**

> 抽取规则的类型。默认值是`SelectorType.XPath`。

可选值如下：

- `SelectorType.XPath` 一般针对html网页或xml，查看[教程](http://www.w3school.com.cn/xpath/index.asp)
- `SelectorType.JsonPath` 针对json数据，查看[教程](http://www.cnblogs.com/draem0507/p/5111002.html)
- `SelectorType.Regex` 可以针对一切文本，查看[教程](http://www.runoob.com/regexp/regexp-tutorial.html)

### selector

**String类型**

> 抽取规则。

**注意：**
如果`selector`为空或者未设置，则抽取的值为`null`，在进行`required`的判定之前，仍会进行`afterExtractField`回调。

### required

**布尔类型**

> 标识当前抽取项的值是否必须（不能为空）。默认是`false`，可以为空。

在抽取过程中，如果某个不能为空值的抽取项，抽取出来的结果是空值，则当前网页的数据抽取会立即结束，抽取结果会被丢弃。

### repeated

**布尔类型**

> 标识当前抽取项的值是否是数组结构。默认是`false`。

如果一个抽取项`repeated`值为`true`，我们称此抽取项是repeated的；如果值为`false`，我们称此抽取项不是repeated的。

**注意：**

1. 抽取规则可以抽取多条，但是`repeated`是`false`，最终的值是抽取结果里面的第一个。
2. 抽取规则只抽取出一条，但是`repeated`是`true`，那么抽取结果还是数组，数组中只包含一个元素。

### children

**field对象数组**

> 抽取项的子抽取项。

field支持子项，可以设置多层级，方便数据以本身的层级方式存储，而不用全部展开到第一层级。

**注意：**
第一层field默认从当前网页内容中抽取，而子项默认从父项的内容中抽取。

### primaryKey

**布尔类型**

> 当前抽取项是否作为整条数据的主键组成部分。默认是`false`。

此主键类似数据库的主键，主要用来标识一条数据。在数据去重以及数据多版本中发挥着重要作用。

两条数据，如果所有被标识为`primaryKey`的字段都有相同的值，则认为这两条数据是同一条数据的两个不同版本。

如果所有字段都未设置`primaryKey`，则默认当前网页链接是`primaryKey`。单页面多数据是个例外，因为有相同的网页链接，所以默认取第一个字段作为`primaryKey`。

### sourceType

**枚举类型**

> 数据抽取源。无默认值，不设置时，抽取源默认时当前网页或父项内容。

可选值：

- `SourceType.UrlContext` 从当前链接的附加内容中抽取
- `SourceType.AttachedUrl` 从attachedUrl的下载内容中抽取

根据`selector`抽取数据时，第一层的field默认从当前网页中抽取，子field默认从其父项的内容中抽取。但是爬取过程中，经常有一些内容不在当前网页中，这时可以通过`sourceType`来设置。

#### SourceType.UrlContext

表示从当前链接的附加内容中抽取数据。在添加链接的时候，可以同时给该链接附加一些数据，通常的使用场景是，列表页展示的一些内容页没有的数据，那么在做链接发现的时候，可以直接把这部分数据附加到对应的内容页链接上。查看对应的示例demo：[爬取列表页数据的Demo1-87870 VR资讯](https://web.archive.org/web/20180904094654/https://www.shenjian.io/index.php?r=demo/docs&demo_id=500004)。

#### SourceType.AttachedUrl

表示需要的数据在另外一个链接（我们叫attachedUrl）的请求结果里面，需要额外再发一次请求来获取数据。

只有当`sourceType`为`SourceType.AttachedUrl`时，下面的`attachedUrl`、`attachedMethod`、`attachedParams`、`attachedHeaders`设置才有意义。

attachedUrl的例子，请查看[评价爬虫Demo-蘑菇街商品及评价](https://web.archive.org/web/20180904094654/https://www.shenjian.io/index.php?r=demo/docs&demo_id=500006)。

多说两句：在还没有`site.requestUrl`接口的时候，很多复杂的场景（比如文章内容分页）只能用`attachedUrl`来实现；相比于`attachedUrl`，`site.requestUrl`具有更高的灵活性，所以对于复杂的场景，我们建议在回调函数中通过`site.requestUrl`来实现，`attachedUrl`只用来实现一些简单的单请求场景。

### attachedUrl

**String类型**

> attachedUrl请求地址。

attachedUrl支持变量替换，变量可引用上下文中已经抽取的字段。

1. 同一层级的field或者第一层级的field，引用方式为”{fieldName}”。
2. 不同层级需要从根field开始，`$`表示根，引用方式为”{$.fieldName.fieldName}”。
3. 特殊变量`$$url`表示当前网页的url，引用方式为”{$$url}”。

比如抽取到字段item_id，attachedUrl形式为`https://item.taobao.com/item.htm?id=1000`，则attachedUrl可以写为：
`"https://item.taobao.com/item.htm?id={item_id}"`

### attachedMethod

**String类型**

> HTTP请求是”GET”还是”POST”。默认是”GET”。

### attachedParams

**String类型**

> HTTP请求的POST参数。如果请求是”GET”，参数将会被忽略。

参数形如`a=b&c=d`，支持变量替换。与attachedUrl的变量引用方式相同。

### attachedHeaders

**对象**

> HTTP请求的headers。

headers不支持变量替换，只能填写固定的值。

### transient

**布尔类型**

> 抽取项是否是临时的。默认是`false`。临时的抽取项，数据存储的时候，不会存储其值。

### type

**String类型**

> 标识抽取项的值类型。默认是`string`。支持的类型包括：`int`、`float`、`image`、`timestamp`、`url`、`string`、`html`、`json`、`bool`。

**注意：**

1. 建议对确定的抽取项标识类型，平台后面会针对类型做优化。
2. 定义字段类型，并不影响数据的存储，更多的是对数据的后期处理（包括预览、发布、导出等），比如类型是`timestamp`的字段，预览时会格式化成日期时间字符串。
3. 爬取结果在搜索和排序时对`int`类型有做优化，如果是整数，建议标识为`int`。
4. mongodb在发布时已经支持了对`int`的转换。