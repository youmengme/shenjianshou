# shenjian对象

shenjian对象是一个全局对象，主要提供一些与神箭手平台相关的函数。

### readSource

```
function readSource(sourceId, query)
```

> `@param` `整数` `sourceId` 数据源ID
>
> `@param` `String` `query` GraphQL的query查询语句，参考[query语法说明](https://web.archive.org/web/20180614212545/http://docs.shenjian.io/develop/platform/graphql/query.html)
>
> `@return` `JS对象` 返回一个数据迭代器对象source，该对象提供`next`和`nextBatch`方法来遍历数据。

该函数提供读取神箭手平台上数据源的能力，通过返回的迭代器对象`source`来遍历数据。内部实现是封装了GraphQL。

#### source.next

```
function next()
```

> `@return` `JS对象` 读取到的下一条数据。没有下一条数据时，返回`undefined`作为结束的判断；异常（如超时）时，返回`null`，一般需要重试。

#### source.nextBatch

```
function nextBatch(size)
```

> `@param` `Integer` `size` 批量获取数据的条数
>
> `@return` `JS对象数组` 读取到的数据数组。没有下一条数据时，返回`undefined`作为结束的判断；异常（如超时）时，返回`null`，一般需要重试。

**注意：**

此函数虽然提供了`size`参数，但是实际返回的数据条数仍然受套餐限制，频率不限，条数限制参考[GraphQL调用限制](https://web.archive.org/web/20180614212545/http://docs.shenjian.io/develop/platform/graphql/limits.html)

示例代码片段：（在initCrawl中读取数据源的数据并生成scanUrl）

```
var sourceId = 14566;//此ID需要修改为您自己的数据源ID
//GraphQL查询语句，查询"city"值为"北京"的所有数据的"shop_id"字段
var query = 'source(city:{eq:"北京"}){data{shop_id}}';
var src = shenjian.readSource(sourceId, query);

configs.initCrawl = function(site) {
  if (src) {
    // 调用"src"的"nextBatch"函数
    // 得到包含10条"shop_id"的JS对象数组
    // "shops"的值是:
    // [{"shop_id":"10041553"},{"shop_id":"10036329"}]
    var shops = src.nextBatch(10);
    for (var s in shops) {
      var shopId = shops[s].shop_id;
      var scanUrl = "https://shop"+shopId+".taobao.com/";
      site.addScanUrl(scanUrl);
    }
  }
};
```