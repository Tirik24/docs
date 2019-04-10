---
title: 火币 API 文档

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.hbg.com/zh-cn/apikey/'>创建 API Key </a>
includes:

search: False
---

# 简介

## API 简介

欢迎使用火币 API！ 你可以使用此 API 获得市场行情数据，进行交易，并且管理你的账户。

在文档的右侧是代码示例，目前我们仅提供针对 `shell` 的代码示例。

你可以通过选择上方下拉菜单的版本号来切换文档对应的 API 版本，也可以通过点击右上方的语言按钮来切换文档语言。

<aside class="notice">
在使用中如果遇到问题，请加技术讨论 QQ 群: 火币网API交流群(5) 778160723（加群时请注明 UID 和编程语言），我们将尽力帮您答疑解惑。
</aside>

## 做市商项目

<aside class="notice">
做市商项目不支持点卡抵扣、VIP、交易量相关活动以及任何形式的返佣活动。
</aside>

欢迎有优秀 maker 策略且交易量大的用户参与长期做市商项目。如果您的火币现货账户或者合约账户中有折合大于20BTC资产（币币和合约账户分开统计），请提供以下信息发送邮件至：

- [MM_service@huobi.com](mailto:MM_service@huobi.com) Huobi Global（现货 / 杠杆）做市商申请；
- [dm_mm@huobi.com](mailto:dm_mm@huobi.com) HBDM（合约）做市商申请。


1. 提供 UID （需不存在返佣关系的 UID）；
2. 提供其他交易平台 maker 交易量截图证明（比如30天内成交量，或者 VIP 等级等）；
3. 请简要阐述做市方法，不需要细节。

## 子账号

子账号可以用来隔离资产与交易，资产可以在母子账号之间划转； 子账号用户只能在子账号内进行交易，并且子账号之间资产不能直接划转，只有母账号有划转权限。

子账号拥有独立的登陆账号密码和 API Key。

<aside class="notice">
子账号的 API Key 目前不能绑定 IP 地址, 所以有效期为90天，过期之后需要重新创建 API Key。
</aside>

子账号可以访问所有公共接口，包括基本信息和市场行情，子账号可以访问的私有接口如下：

接口|说明|
----------------------|---------------------|
[POST /v1/order/orders/place](#fd6ce2a756)	|创建并执行订单|
[POST /v1/order/orders/{order-id}/submitcancel](#4e53c0fccd)	|撤销一个订单|
[POST /v1/order/orders/batchcancel](#ad00632ed5)	|批量撤销订单|
[POST /v1/order/orders/batchCancelOpenOrders](#open-orders)	|撤销当前委托订单|
[GET /v1/order/orders/{order-id}](#92d59b6aad)	|查询一个订单详情|
[GET /v1/order/orders](#d72a5b49e7)	|查询当前委托、历史委托|
[GET /v1/order/openOrders](#95f2078356)	|查询当前委托订单|
[GET /v1/order/matchresults](#0fa6055598)	|查询成交|
[GET /v1/order/orders/{order-id}/matchresults](#56c6c47284)	|查询某个订单的成交明细|
[GET /v1/account/accounts](#bd9157656f)	|查询当前用户的所有账户|
[GET /v1/account/accounts/{account-id}/balance](#870c0ab88b)	|查询指定账户的余额|

<aside class="notice">
其他接口子账号不可访问，如果尝试访问，系统会返回 “error-code 403”。
</aside>

# 更新日志

|  生效时间（北京时间 UTC+8) | 接口 | 新增 / 修改 | 摘要 |
|-----|-----|-----|-----|
| 2019.01.17 07:00 | - Websocket accounts           | 修改 | - 增加订阅参数 model；<br>- 订阅返回的内容中不再推送交易子账户冻结余额的变化。 |
| 2018.07.10 11:00 | - GET `/market/history/kline`  | 修改 | - `size` 取值范围由 [1-1000] 修改为 [1-2000]。|
| 2018.07.06 16:00 | - POST `/v1/order/orders/place`| 修改 | - 添加 `buy-limit-maker`，`sell-limit-maker` 两种下单类型支持；<br>- 新增获取某个帐号下指定交易对或者所有交易对的所有尚未成交订单接口: `/v1/order/openOrders`。
| 2018.07.06 16:00 | - GET `/v1/order/openOrders`<br>- POST `/v1/order/orders/batchCancelOpenOrders` | 新增 | - 新增获取某个帐号下指定交易对或者所有交易对的所有尚未成交订单接口；<br>- 新增批量取消某个帐号下指定的订单列表中所有订单接口。 |
| 2018.07.02 16:00 | - ETF 相关接口 | 新增 | - 本次接口变更主要是支持 HB10 ETF 的换入和换出。 |
| 2018.06.20 16:00 | - GET `/market/tickers` | 新增 | - 新增 Tickers 接口，Tickers 为当前所有交易对行情数据。 |


# 接入说明

## 接入 URLs


**REST API**

**`https://api.huobi.pro`**

**Websocket Feed（行情）**

**`wss://api.huobi.pro/ws`**

**Websocket Feed（资产和订单）**

**`wss://api.huobi.pro/ws/v1`**

<aside class="notice">
请使用中国大陆以外的 IP 访问火币 API。
</aside>
<aside class="notice">
鉴于延迟高和稳定性差等原因，不建议通过代理的方式访问火币 API。
</aside>

## 限频规则

- 现货 / 杠杆（api.huobi.pro）：10秒100次

<aside class="notice">
单个 API Key 维度限制，建议行情 API 访问也要加上签名，否则限频会更严格。
</aside>


## 签名认证

### 签名说明

API 请求在通过 internet 传输的过程中极有可能被篡改，为了确保请求未被更改，除公共接口（基础信息，行情数据）外的私有接口均必须使用您的 API Key 做签名认证，以校验参数或参数值在传输途中是否发生了更改。

一个合法的请求由以下几部分组成：

- 方法请求地址：即访问服务器地址 api.huobi.pro，比如 api.huobi.pro/v1/order/orders。

- API 访问密钥（AccessKeyId）：您申请的 API Key 中的 Access Key。

- 签名方法（SignatureMethod）：用户计算签名的基于哈希的协议，此处使用 HmacSHA256。

- 签名版本（SignatureVersion）：签名协议的版本，此处使用2。

- 时间戳（Timestamp）：您发出请求的时间 (UTC 时区) (UTC 时区) (UTC 时区) 。如：2017-05-11T16:22:06。在查询请求中包含此值有助于防止第三方截取您的请求。

- 必选和可选参数：每个方法都有一组用于定义 API 调用的必需参数和可选参数。可以在每个方法的说明中查看这些参数及其含义。 请一定注意：对于 GET 请求，每个方法自带的参数都需要进行签名运算； 对于 POST 请求，每个方法自带的参数不进行签名认证，即 POST 请求中需要进行签名运算的只有 AccessKeyId、SignatureMethod、SignatureVersion、Timestamp 四个参数，其它参数放在 body 中。

- 签名：签名计算得出的值，用于确保签名有效和未被篡改。


### 创建 API Key

您可以在 <a href='https://www.hbg.com/zh-cn/apikey/'>这里 </a> 创建 API Key。

API Key 包括以下两部分

- `Access Key`  API 访问密钥
  
- `Secret Key`  签名认证加密所使用的密钥（仅申请时可见）

<aside class="notice">
创建 API Key 时可以选择绑定 IP 地址，未绑定 IP 地址的 API Key 有效期为90天。
</aside>
<aside class="notice">
API Key 具有包括交易、借贷和充提币等所有操作权限。
</aside>
<aside class="warning">
这两个密钥与账号安全紧密相关，无论何时都请勿向其它人透露。
</aside>


### 签名步骤

规范要计算签名的请求 因为使用 HMAC 进行签名计算时，使用不同内容计算得到的结果会完全不同。所以在进行签名计算前，请先对请求进行规范化处理。下面以查询某订单详情请求为例进行说明：

查询某订单详情

`https://api.huobi.pro/v1/order/orders?`

`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx`

`&SignatureMethod=HmacSHA256`

`&SignatureVersion=2`

`&Timestamp=2017-05-11T15:19:30`

`&order-id=1234567890`

#### 1. 请求方法（GET 或 POST），后面添加换行符 “\n”


`GET\n`

#### 2. 添加小写的访问地址，后面添加换行符 “\n”

`
api.huobi.pro\n
`

#### 3. 访问方法的路径，后面添加换行符 “\n”

`
/v1/order/orders\n
`

#### 4. 按照ASCII码的顺序对参数名进行排序。例如，下面是请求参数的原始顺序，进行过编码后


`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx`

`order-id=1234567890`

`SignatureMethod=HmacSHA256`

`SignatureVersion=2`

`Timestamp=2017-05-11T15%3A19%3A30`

<aside class="notice">
使用 UTF-8 编码，且进行了 URI 编码，十六进制字符必须大写，如 “:” 会被编码为 “%3A” ，空格被编码为 “%20”。
</aside>
<aside class="notice">
时间戳（Timestamp）需要以YYYY-MM-DDThh:mm:ss格式添加并且进行 URI 编码。
</aside>


#### 5. 经过排序之后

`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx`

`SignatureMethod=HmacSHA256`

`SignatureVersion=2`

`Timestamp=2017-05-11T15%3A19%3A30`

`order-id=1234567890`

#### 6. 按照以上顺序，将各参数使用字符 “&” 连接


`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890`

#### 7. 组成最终的要进行签名计算的字符串如下

`GET\n`

`api.huobi.pro\n`

`/v1/order/orders\n`

`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890`


#### 8. 用上一步里生成的 “请求字符串” 和你的密钥 (Secret Key) 生成一个数字签名

`4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=`

1. 将上一步得到的请求字符串和 API 私钥作为两个参数，调用HmacSHA256哈希函数来获得哈希值。

2. 将此哈希值用base-64编码，得到的值作为此次接口调用的数字签名。

#### 9. 将生成的数字签名加入到请求的路径参数里

最终，发送到服务器的 API 请求应该为

`https://api.huobi.pro/v1/order/orders?AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&order-id=1234567890&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&Signature=4F65x5A2bLyMWVQj3Aqp%2BB4w%2BivaA7n5Oi2SuYtCJ9o%3D`

1. 把所有必须的认证参数添加到接口调用的路径参数里

2. 把数字签名在URL编码后加入到路径参数里，参数名为“Signature”。

## 请求格式

所有的API请求都以GET或者POST形式发出。对于GET请求，所有的参数都在路径参数里；对于POST请求，所有参数则以JSON格式发送在请求主体（body）里。

## 返回格式

所有的接口返回都是JSON格式。在JSON最上层有几个表示请求状态和属性的字段："status", "ch", 和 "ts". 实际的接口返回内容在"data"字段里.

### 返回内容格式

> 返回内容将会是以下格式:

```json
{
  "status": "ok",
  "ch": "market.btcusdt.kline.1day",
  "ts": 1499223904680,
  "data": // per API response data in nested JSON object
}
```

参数名称| 数据类型 | 描述
--------- | --------- | -----------
status    | string    | API接口返回状态
ch        | string    | 接口数据对应的数据流。部分接口没有对应数据流因此不返回此字段
ts        | int       | 接口返回的调整为北京时间的时间戳，单位毫秒
data      | object    | 接口返回数据主体

## 错误信息

### 行情 API 错误信息

| 错误码  |  描述 |
|-----|-----|
| bad-request | 错误请求 |
| invalid-parameter | 参数错误 |
| invalid-command | 指令错误 |
code 的具体解释, 参考对应的 `err-msg`.

### 交易 API 错误信息

| 错误码  |  描述 |
|-----|-----|
| base-symbol-error |  交易对不存在 |
| base-currency-error |  币种不存在 |
| base-date-error | 错误的日期格式 |
| account for id `12,345` and user id `6,543,210` does not exist| `account-id` 错误，请使用GET `/v1/account/accounts` 接口查询 | 
| account-frozen-balance-insufficient-error | 余额不足 |
| account-transfer-balance-insufficient-error | 余额不足无法冻结 |
| bad-argument | 无效参数 |
| api-signature-not-valid | API签名错误 |
| gateway-internal-error | 系统繁忙，请稍后再试|
| ad-ethereum-addresss| 请输入有效的以太坊地址|
| order-accountbalance-error| 账户余额不足|
| order-limitorder-price-error|限价单下单价格超出限制 |
|order-limitorder-amount-error|限价单下单数量超出限制 |
|order-orderprice-precision-error|下单价格超出精度限制 |
|order-orderamount-precision-error|下单数量超过精度限制|
|order-marketorder-amount-error|下单数量超出限制|
|order-queryorder-invalid|查询不到此条订单|
|order-orderstate-error|订单状态错误|
|order-datelimit-error|查询超出时间限制|
|order-update-error|订单更新出错|

##  SDK 与代码示例

**SDK**

[Java](https://github.com/huobiapi/huobi_Java)


**Websocket**

[Python3](https://github.com/huobiapi/Websocket-Python3-demo)

[Node.js](https://github.com/huobiapi/WebSocket-Node.js-demo)

[PHP](https://github.com/huobiapi/WebSocket-PHP-demo)

**REST**

[Python3](https://github.com/huobiapi/REST-Python3-demo)

[Java](https://github.com/huobiapi/REST-Java-demo)

[Node.js](https://github.com/huobiapi/REST-Node.js-demo)

[C#](https://github.com/huobiapi/REST-CSharp-demo)

[go](https://github.com/huobiapi/REST-GO-demo)

[PHP](https://github.com/huobiapi/REST-PHP-demo)

[C++](https://github.com/huobiapi/REST-Cpp-demo)

[Objective-C](https://github.com/huobiapi/REST-ObjectiveC-demo)

[QTC++](https://github.com/huobiapi/REST-QTCpp-demo)

[Python2.7](https://github.com/huobiapi/REST-Python2.7-demo)

[Ruby](https://github.com/huobiapi/REST-Ruby-demo)

[易语言](https://github.com/huobiapi/REST-YiYuyan-demo)

## 常见问题 Q & A

### 经常断线或者丢数据

* 请确认是否使用 api.huobi.pro 域名访问火币 API
* 请使用日本云服务器

### 签名失败

* 检查 API Key 是否有效，是否复制正确，是否有绑定 IP 白名单
* 检查时间戳是否是 UTC 时间
* 检查参数是否按字母排序
* 检查编码
* 检查签名是否有 base64 编码
* 检查 GET 是否以表单方式提交
* 检查 POST 的 url 是否带着签名字段，POST 的数据格式是否是 json 格式
* 检查签名结果是否有进行 URI 编码

### 返回 login-required

* 检查参数 `account-id` 是否是由 GET `/v1/account/accounts` 接口返回的，而不是填的 UID
* 检查是否 POST 请求是否把业务参数也计算进签名
* 检查 GET 请求是否将参数按照 ASCII 码表顺序排序

### 返回 gateway-internal-error

* 检查 POST 请求是否在 header 中声明 Content-Type:application/json




# 基础信息

## 获取所有交易对

此接口返回所有火币全球站支持的交易对。

```shell
curl "https://api.huobi.pro/v1/common/symbols"
```


### HTTP 请求

- GET `/v1/common/symbols`

### 请求参数

此接口不接受任何参数。

> Responds:

```json
  "data": [
    {
        "base-currency": "btc",
        "quote-currency": "usdt",
        "price-precision": 2,
        "amount-precision": 4,
        "symbol-partition": "main",
        "symbol": "btcusdt"
    }
    {
        "base-currency": "eth",
        "quote-currency": "usdt",
        "price-precision": 2,
        "amount-precision": 4,
        "symbol-partition": "main",
        "symbol": "ethusdt"
    }
  ]
```

### 返回字段

字段名称            | 数据类型 | 描述
---------       | --------- | -----------
base-currency   | string    | 交易对中的基础币种
quote-currency  | string    | 交易对中的报价币种
price-precision | integer   | 交易对报价的精度（小数点后位数）
amount-precision| integer   | 交易对基础币种计数精度（小数点后位数）
symbol-partition| string    | 交易区，可能值: [main，innovation，bifurcation]

## 获取所有币种

此接口返回所有火币全球站支持的币种。


```shell
curl "https://api.huobi.pro/v1/common/currencys"
```

### HTTP 请求

- GET `/v1/common/currencys`


### 请求参数

此接口不接受任何参数。


> Response:

```json
  "data": [
    "usdt",
    "eth",
    "etc"
  ]
```

### 返回字段


<aside class="notice">返回的“data”对象是一个字符串数组，每一个字符串代表一个支持的币种。</aside>


## 获取当前系统时间

此接口返回当前的系统时间，时间是调整为北京时间的时间戳，单位毫秒。

```shell
curl "https://api.huobi.pro/v1/common/timestamp"
```

### HTTP 请求

- GET `/v1/common/timestamp`

### 请求参数

此接口不接受任何参数。

> Response:

```json
  "data": 1494900087029
```

### 响应数据

返回的“data”对象是一个整数表示返回的调整为北京时间的时间戳，单位毫秒。

# 行情数据

## K 线数据（蜡烛图）

此接口返回历史K线数据。

### HTTP 请求

- GET `/market/history/kline`

```shell
curl "https://api.huobi.pro/market/history/kline?period=1day&size=200&symbol=btcusdt"
```

### 请求参数

参数       | 数据类型 | 是否必须 | 默认值 | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------ | ------
symbol    | string    | true     | NA      | 交易对 | btcusdt, ethbtc...
period    | string    | true     | NA      | 返回数据时间粒度，也就是每根蜡烛的时间区间 | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year
size      | integer   | false    | 150     | 返回 K 线数据条数 | [1, 2000]

<aside class="notice">当前 REST API 不支持自定义时间区间，如需要历史固定时间范围的数据，请参考 Websocket API 中的 K 线接口。</aside>

<aside class="notice">获取 hb10 净值时， symbol 请填写 “hb10”。</aside>

> Response:

```json
[
  {
    "id": 1499184000,
    "amount": 37593.0266,
    "count": 0,
    "open": 1935.2000,
    "close": 1879.0000,
    "low": 1856.0000,
    "high": 1940.0000,
    "vol": 71031537.97866500
  }
]
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | 调整为北京时间的时间戳，单位秒，并以此作为此K线柱的id
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易次数
open      | float     | 本阶段开盘价
close     | float     | 本阶段收盘价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 以报价币种计量的交易量



## 聚合行情（Ticker）

此接口获取ticker信息同时提供最近24小时的交易聚合信息。

### HTTP 请求

- GET `/market/detail/merged`

```shell
curl "https://api.huobi.pro/market/detail/merged?symbol=ethusdt"
```

### 请求参数

参数      | 数据类型   | 是否必须  | 默认值  | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------| -----
symbol    | string    | true     | NA      | 交易对 | btcusdt, ethbtc


> Response:

```json
{
  "id":1499225271,
  "ts":1499225271000,
  "close":1885.0000,
  "open":1960.0000,
  "high":1985.0000,
  "low":1856.0000,
  "amount":81486.2926,
  "count":42122,
  "vol":157052744.85708200,
  "ask":[1885.0000,21.8804],
  "bid":[1884.0000,1.6702]
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | NA
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易次数
open      | float     | 本阶段开盘价
close     | float     | 本阶段最新价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 以报价币种计量的交易量
bid       | object    | 当前的最高卖价 [price, quote volume]
ask       | object    | 当前的最低买价 [price, quote volume]


## 所有交易对的最新 Tickers

获得所有交易对的 tickers，数据取值时间区间为24小时滚动。

<aside class="notice">此接口返回所有交易对的 ticker，因此数据量较大。</aside>

### HTTP 请求

- GET `/market/tickers`

```shell
curl "https://api.huobi.pro/market/tickers"
```



### 请求参数

此接口不接受任何参数。

> Response:

```json
[  
    {  
        "open":0.044297,      // 开盘价
        "close":0.042178,     // 收盘价
        "low":0.040110,       // 最高价
        "high":0.045255,      // 最低价
        "amount":12880.8510,  
        "count":12838,
        "vol":563.0388715740,
        "symbol":"ethbtc"
    },
    {  
        "open":0.008545,
        "close":0.008656,
        "low":0.008088,
        "high":0.009388,
        "amount":88056.1860,
        "count":16077,
        "vol":771.7975953754,
        "symbol":"ltcbtc"
    }
]
```

### 响应数据

核心响应数据为一个对象列，每个对象包含下面的字段

字段名称      | 数据类型   | 描述
--------- | --------- | -----------
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易笔数
open      | float     | 开盘价
close     | float     | 最新价
low       | float     | 最低价
high      | float     | 最高价
vol       | float     | 以报价币种计量的交易量
symbol    | string    | 交易对，例如btcusdt, ethbtc

## 市场深度数据

此接口返回指定交易对的当前市场深度数据。

### HTTP 请求

- GET `/market/depth`

```shell
curl "https://api.huobi.pro/market/depth?symbol=btcusdt&type=step2"
```

### 请求参数

参数      | 数据类型   | 必须     | 默认值 | 描述 | 取值范围 |
--------- | --------- | -------- | ------| ---- | --- |
symbol    | string    | true     | NA    | 交易对 | btcusdt, ethbtc...|
depth     | integer   | false    | 20    | 返回深度的数量 | 5，10，20 |
type      | string    | true     | step0 | 深度的价格聚合度，具体说明见下方 | step0，step1，step2，step3，step4，step5 |

<aside class="notice">当type值为‘step0’时，‘depth’的默认值为150而非20。 </aside>

**参数type的各值说明**

取值      | 说明
--------- | ---------
step0     | 无聚合
step1     | 聚合度为报价精度*10
step2     | 聚合度为报价精度*100
step3     | 聚合度为报价精度*1000
step4     | 聚合度为报价精度*10000
step5     | 聚合度为报价精度*100000

> Response:

```json
{
    "version": 31615842081,
    "ts": 1489464585407,
    "bids": [
      [7964, 0.0678], // [price, amount]
      [7963, 0.9162],
      [7961, 0.1],
      [7960, 12.8898],
      [7958, 1.2],
      ...
    ],
    "asks": [
      [7979, 0.0736],
      [7980, 1.0292],
      [7981, 5.5652],
      [7986, 0.2416],
      [7990, 1.9970],
      ...
    ]
  }
```

### 响应数据

<aside class="notice">返回的JSON顶级数据对象名为'tick'而不是通常的'data'。</aside>

字段名称      | 数据类型    | 描述
--------- | --------- | -----------
ts        | integer   | 调整为北京时间的时间戳，单位毫秒
version   | integer   | 内部字段
bids      | object    | 当前的所有买单 [price, quote volume]
asks      | object    | 当前的所有卖单 [price, quote volume]

## 最近市场成交记录

此接口返回指定交易对最新的一个交易记录。

### HTTP 请求

- GET `/market/trade`

```shell
curl "https://api.huobi.pro/market/trade?symbol=ethusdt"
```

### 请求参数

参数      | 数据类型   | 是否必须  | 默认值   | 描述
--------- | --------- | -------- | ------- | -----------
symbol    | string    | true     | NA      | 交易对，例如btcusdt, ethbtc

> Response:

```json
{
    "id": 600848670,
    "ts": 1489464451000,
    "data": [
      {
        "id": 600848670,
        "price": 7962.62,
        "amount": 0.0122,
        "direction": "buy",
        "ts": 1489464451000
      }
    ]
}
```

### 响应数据

<aside class="notice">返回的JSON顶级数据对象名为'tick'而不是通常的'data'。</aside>

字段名称       | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | 唯一交易id
amount    | float     | 以基础币种为单位的交易量
price     | float     | 以报价币种为单位的成交价格
ts        | integer   | 调整为北京时间的时间戳，单位毫秒
direction | string    | 交易方向：“buy” 或 “sell”, “buy” 即买，“sell” 即卖

## 获得近期交易记录

此接口返回指定交易对近期的所有交易记录。

### HTTP 请求

- GET `/market/history/trade`

```shell
curl "https://api.huobi.pro/market/history/trade?symbol=ethusdt&size=2"
```

### 请求参数

参数       | 数据类型  | 是否必须   | 默认值 | 描述
--------- | --------- | -------- | ------- | -----------
symbol    | string    | true     | NA      | 交易对，例如 btcusdt, ethbtc
size      | integer   | false    | 1       | 返回的交易记录数量，最大值2000

> Response:

```json
[  
   {  
      "id":31618787514,
      "ts":1544390317905,
      "data":[  
         {  
            "amount":9.000000000000000000,
            "ts":1544390317905,
            "id":3161878751418918529341,
            "price":94.690000000000000000,
            "direction":"sell"
         },
         {  
            "amount":73.771000000000000000,
            "ts":1544390317905,
            "id":3161878751418918532514,
            "price":94.660000000000000000,
            "direction":"sell"
         }
      ]
   },
   {  
      "id":31618776989,
      "ts":1544390311353,
      "data":[  
         {  
            "amount":1.000000000000000000,
            "ts":1544390311353,
            "id":3161877698918918522622,
            "price":94.710000000000000000,
            "direction":"buy"
         }
      ]
   }
]
```

### 响应数据

<aside class="notice">返回的数据对象是一个对象数组，每个数组元素为一个调整为北京时间的时间戳（单位毫秒）下的所有交易记录，这些交易记录以数组形式呈现。</aside>

参数      | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | 唯一交易id
amount    | float     | 以基础币种为单位的交易量
price     | float     | 以报价币种为单位的成交价格
ts        | integer   | 调整为北京时间的时间戳，单位毫秒
direction | string    | 交易方向：“buy” 或 “sell”, “buy” 即买，“sell” 即卖

## 最近24小时行情数据

此接口返回最近24小时的行情数据汇总。

### HTTP 请求

- GET `/market/detail`

```shell
curl "https://api.huobi.pro/market/detail?symbol=ethusdt"
```

### 请求参数

参数      | 数据类型 | 是否必须 | 默认值 | 描述
--------- | --------- | -------- | ------- | -----------
symbol    | string    | true     | NA      | 交易对，例如btcusdt, ethbtc

> Response:

```json
{  
   "amount":613071.438479561,
   "open":86.21,
   "close":94.35,
   "high":98.7,
   "id":31619471534,
   "count":138909,
   "low":84.63,
   "version":31619471534,
   "vol":5.6617373443873316E7
}
```

### 响应数据

<aside class="notice">返回的JSON顶级数据对象名为'tick'而不是通常的'data'。</aside>

字段名称      | 数据类型   | 描述
--------- | --------- | -----------
id        | integer   | 响应id
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易次数
open      | float     | 本阶段开盘价
close     | float     | 本阶段收盘价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 以报价币种计量的交易量
version   | integer   | 内部数据

# 账户相关

<aside class="notice">访问账户相关的接口需要进行签名认证。</aside>

## 账户信息

查询当前用户的所有账户 ID `account-id` 及其相关信息

### HTTP 请求

- GET `/v1/account/accounts`

### 请求参数

无

> Response:

```json
{
  "data": [
    {
      "id": 100001,
      "type": "spot",
      "subtype": "",
      "state": "working"
    }
    {
      "id": 100002,
      "type": "margin",
      "subtype": "btcusdt",
      "state": "working"
    },
    {
      "id": 100003,
      "type": "otc",
      "subtype": "",
      "state": "working"
    }
  ]
}
```

### 响应数据

| 参数名称  | 是否必须 | 数据类型 | 描述 | 取值范围 |
| ----- | ---- | ------ | ----- | ----  |
| id    | true | long   | account-id |    |
| state | true | string | 账户状态  | working：正常, lock：账户被锁定 |
| type  | true | string | 账户类型  | spot：现货账户， margin：杠杆账户，otc：OTC 账户，point：点卡账户  |

<aside class="notice">杠杆账户（margin）会在第一次划转资产时创建，如果未划转过资产则不会有杠杆账户。</aside>

## 账户余额

查询指定账户的余额，支持以下账户：

spot：现货账户， margin：杠杆账户，otc：OTC 账户，point：点卡账户

### HTTP 请求

- GET `/v1/account/accounts/{account-id}/balance`

### 请求参数

| 参数名称   | 是否必须 | 类型   | 描述   | 默认值  | 取值范围 |
| ---------- | ---- | ------ | --------------- | ---- | ---- |
| account-id | true | string | account-id，填在 path 中，可用 GET /v1/account/accounts 获取 |  |      |

> Response:

```json
{
  "data": {
    "id": 100009,
    "type": "spot",
    "state": "working",
    "list": [
      {
        "currency": "usdt",
        "type": "trade",
        "balance": "5007.4362872650"
      },
      {
        "currency": "usdt",
        "type": "frozen",
        "balance": "348.1199920000"
      }
    ],
    "user-id": 10000
  }
}
```

### 响应数据

| 参数名称  | 是否必须  | 数据类型   | 描述    | 取值范围   |
| ----- | ----- | ------ | ----- | ----- |
| id    | true  | long   | 账户 ID |      |
| state | true  | string | 账户状态  | working：正常  lock：账户被锁定 |
| type  | true  | string | 账户类型  | spot：现货账户， margin：杠杆账户，otc：OTC 账户，point：点卡账户 |
| list  | false | Array  | 子账号数组 |     |

list字段说明

| 参数名称   | 是否必须 | 数据类型   | 描述   | 取值范围    |
| -------- | ---- | ------ | ---- |  ------ |
| balance  | true | string | 余额   |    |
| currency | true | string | 币种   |    |
| type     | true | string | 类型   | trade: 交易余额，frozen: 冻结余额 |

## 资产划转（母子账号之间）

母账户执行母子账号之间的划转

### HTTP 请求

- POST ` /v1/subuser/transfer`

### 请求参数

参数|是否必填 | 数据类型 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--
sub-uid	|true|	long|子账号uid	|-|
currency|true|	string|币种	|-|
amount|true|	decimal|划转金额|-|	
type|true|string|划转类型| master-transfer-in（子账号划转给母账户虚拟币）/ master-transfer-out （母账户划转给子账号虚拟币）/master-point-transfer-in （子账号划转给母账户点卡）/master-point-transfer-out（母账户划转给子账号点卡） |

> Response:

```json
{
  "data":123456,
  "status":"ok"
}
```

### 响应数据

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
data | true| int | - | 划转订单id|   - |
status | true|   | - |  状态| "OK" or "Error"    |

### 错误码

error_code|	说明|	类型|
------------------|------------|-----------|
account-transfer-balance-insufficient-error|	账户余额不足|	string|
base-operation-forbidden|	禁止操作（母子账号关系错误时报）	|string|

## 子账号余额（汇总）

母账户查询其下所有子账号的各币种汇总余额

### HTTP 请求

- GET `/v1/subuser/aggregate-balance`

### 请求参数

无

> Response:

```json
{
  "status": "ok",
  "data": [
      {
        "currency": "eos",
        "balance": "1954559.809500000000000000"
      },
      {
        "currency": "btc",
        "balance": "0.000000000000000000"
      },
      {
        "currency": "usdt",
        "balance": "2925209.411300000000000000"
      },
      ...
   ]
}
```

### 响应数据

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
status | true|   | - |  状态| "OK" or "Error"    |
data | true| list | - | |   - |

- data 

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
currency|	是|	string|	-|	子账号币名|-|	
balance|	是|	string|	-|	子账号下该币种所有余额（可用余额和冻结余额的总和）|-|

## 子账号余额

母账户查询子账号各币种账户余额

### HTTP 请求

- GET `/v1/account/accounts/{sub-uid}`

### 请求参数

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
sub-uid|true|	long|	-|	子用户的 UID|-|

> Response:

```json
{
  "status": "ok",
	"data": [
    {
      "id": 9910049,
      "type": "spot",
      "list": 
      [
        {
          "currency": "btc",
          "type": "trade",
          "balance": "1.00"
        },
        {
          "currency": "eth",
          "type": "trade",
          "balance": "1934.00"
        }
      ]
    },
    {
      "id": 9910050,
      "type": "point",
      "list": []
    }
	]
}
```

### 响应数据


参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
id|	-	|long|	-	|子账号 UID|-|	
type|	-	|string|	-	|账户类型|	Spot：现货账户，point：点卡账户|
list|	-	|object|	-	|-|-|

- list
	
参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
currency|	-	|string|	-	|币种	|-|
type|	-	|string|	-	|账户类型	|trade：交易账户，frozen：冻结账户|
balance|-|decimal|-		|账户余额	|-|

# 钱包（充值与提现）

<aside class="notice">访问钱包相关的接口需要进行签名认证。</aside>

## 虚拟币提现

<aside class="notice">仅支持在官网上相应币种 <a href='https://www.hbg.com/zh-cn/withdraw_address/'>地址列表 </a> 中的地址。</aside>

### HTTP 请求

- POST ` /v1/dw/withdraw/api/create`

```json
{
  "address": "0xde709f2102306220921060314715629080e2fb77",
  "amount": "0.05",
  "currency": "eth",
  "fee": "0.01"
}
```

### 请求参数

| 参数名称       | 是否必须 | 类型     | 描述     |取值范围 |
| ---------- | ---- | ------ | ------ | ---- |
| address | true | string   | 提现地址 |仅支持在官网上相应币种[地址列表](https://www.hbg.com/zh-cn/withdraw_address/) 中的地址  |
| amount     | true | string | 提币数量   |      |
| currency | true | string | 资产类型   |  btc, ltc, bch, eth, etc ...(火币全球站支持的币种) |
| fee     | false | string | 转账手续费  |     |
| chain   | false | string | 提 USDT-ERC20 时需要设置此参数为 "usdterc20"，其他币种提现不需要设置此参数  |     |
| addr-tag|false | string | 虚拟币共享地址tag，适用于xrp，xem，bts，steem，eos，xmr | 格式, "123"类的整数字符串|


> Response:

```json
{
  "data": 700
}
```

### 响应数据


| 参数名称 | 是否必须  | 数据类型 | 描述   | 取值范围 |
| ---- | ----- | ---- | ---- | ---- |
| data | false | long | 提现 ID |      |


## 取消提现



### HTTP 请求

- POST ` /v1/dw/withdraw-virtual/{withdraw-id}/cancel`

### 请求参数

| 参数名称        | 是否必须 | 类型   | 描述 | 默认值  | 取值范围 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| withdraw-id | true | long | 提现 ID，填在 path 中 |      |      |


> Response:

```json
{
  "data": 700
}
```

### 响应数据


| 参数名称 | 是否必须  | 数据类型 | 描述    | 取值范围 |
| ---- | ----- | ---- | ----- | ---- |
| data | false | long | 提现 ID |      |

## 充提记录


### HTTP 请求

- GET `/v1/query/deposit-withdraw`

### 请求参数

| 参数名称        | 是否必须 | 类型   | 描述 | 默认值  | 取值范围 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| currency | true | string | 币种  |  |  |
| type | true | string | 充值或提现 |     |  deposit 或 withdraw |
| from   | true | string | 查询起始 ID  |    |     |
| size   | true | string | 查询记录大小  |    |     |


> Response:

```json
{
  "data":
    [
      {
        "id": 1171,
        "type": "deposit",
        "currency": "xrp",
        "tx-hash": "ed03094b84eafbe4bc16e7ef766ee959885ee5bcb265872baaa9c64e1cf86c2b",
        "amount": 7.457467,
        "address": "rae93V8d2mdoUQHwBDBdM4NHCMehRJAsbm",
        "address-tag": "100040",
        "fee": 0,
        "state": "safe",
        "created-at": 1510912472199,
        "updated-at": 1511145876575
      },
      ...
    ]
}
```

### 响应数据


| 参数名称 | 是否必须 | 数据类型 | 描述 | 取值范围 |
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |   | |
|   type  |  true  |  string  | 类型 | 'deposit', 'withdraw' |
|   currency  |  true  |  string  |  币种 | |
| tx-hash | true |string | 交易哈希 | |
| amount | true | long | 个数 | |
| address | true | string | 地址 | |
| address-tag | true | string | 地址标签 | |
| fee | true | long | 手续费 | |
| state | true | string | 状态 | 状态参见下表 |
| created-at | true | long | 发起时间 | |
| updated-at | true | long | 最后更新时间 | |


- 虚拟币充值状态定义：

|状态|描述|
|--|--|
|unknown|状态未知|
|confirming|确认中|
|confirmed|确认中|
|safe|已完成|
|orphan| 待确认|

- 虚拟币提现状态定义：

| 状态 | 描述  |
|--|--|
| submitted | 已提交 |
| reexamine | 审核中 |
| canceled  | 已撤销 |
| pass    | 审批通过 |
| reject  | 审批拒绝 |
| pre-transfer | 处理中 |
| wallet-transfer | 已汇出 |
| wallet-reject   | 钱包拒绝 |
| confirmed      | 区块已确认 |
| confirm-error  | 区块确认错误 |
| repealed       | 已撤销 |



# 现货 / 杠杆交易

<aside class="notice">访问交易相关的接口需要进行签名认证。</aside>

<aside class="warning">杠杆交易时，“account-id” 参数需设置为 “margin” 的 account-id， “source”参数需设置为 “margin-api”。</aside>

## 下单

发送一个新订单到火币以进行撮合。

### HTTP 请求

- POST ` /v1/order/orders/place`

```json
{
  "account-id": "100009",
  "amount": "10.1",
  "price": "100.1",
  "source": "api",
  "symbol": "ethusdt",
  "type": "buy-limit"
}
```

### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
account-id | string    | true     | NA      | 账户 ID，使用 GET /v1/account/accounts 接口查询。现货交易使用 ‘spot’ 账户的 account-id；杠杆交易，请使用 ‘margin’ 账户的 account-id
symbol     | string    | true     | NA      | 交易对, 例如btcusdt, ethbtc
type       | string    | true     | NA      | 订单类型，包括buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc, buy-limit-maker, sell-limit-maker（说明见下文）
amount     | string    | true     | NA      | 订单交易量
price      | string    | false    | NA      | limit order的交易价格
source     | string    | false    | api     | 现货交易填写“api”，杠杆交易填写“margin-api”

**buy-limit-maker**

当“下单价格”>=“市场最低卖出价”，订单提交后，系统将拒绝接受此订单；

当“下单价格”<“市场最低卖出价”，提交成功后，此订单将被系统接受。

**sell-limit-maker**

当“下单价格”<=“市场最高买入价”，订单提交后，系统将拒绝接受此订单；

当“下单价格”>“市场最高买入价”，提交成功后，此订单将被系统接受。

> Response:

```json
{  
  "data": "59378"
}
```

### 响应数据

返回的主数据对象是一个对应下单单号的字符串。

## 撤销订单

此接口发送一个撤销订单的请求。

<aside class="warning">此接口只提交取消请求，实际取消结果需要通过订单状态，撮合状态等接口来确认。</aside>


### HTTP 请求

- POST ` /v1/order/orders/{order-id}/submitcancel`


### 请求参数

| 参数名称     | 是否必须 | 类型     | 描述           | 默认值  | 取值范围 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| order-id | true | string | 订单ID，填在path中 |      |      |


> Response:

```json
{  
  "data": "59378"
}
```

### 响应数据

返回的主数据对象是一个对应下单单号的字符串。


## 查询当前未成交订单

查询已提交但是仍未完全成交或未被撤销的订单。


```json
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
```

### HTTP 请求

- GET `/v1/order/openOrders`


### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
account-id | string    | false    | NA      | 账户 ID，使用 GET /v1/account/accounts 接口获得。现货交易使用‘spot’账户的 account-id；杠杆交易，请使用 ‘margin’ 账户的 account-id
symbol     | string    | false    | NA      | 交易对, 例如btcusdt, ethbtc
side       | string    | false    | both    | 指定只返回某一个方向的订单，可能的值有: buy, sell. 默认两个方向都返回。
size       | int       | false    | 10      | 返回订单的数量，最大值2000。

> Response:

```json
{  
  "data": [
    {
      "id": 5454937,
      "symbol": "ethusdt",
      "account-id": 30925,
      "amount": "1.000000000000000000",
      "price": "0.453000000000000000",
      "created-at": 1530604762277,
      "type": "sell-limit",
      "filled-amount": "0.0",
      "filled-cash-amount": "0.0",
      "filled-fees": "0.0",
      "source": "web",
      "state": "submitted"
    }
  ]
}
```

### 响应数据

字段名称          | 数据类型 | 描述
---------           | --------- | -----------
id                  | integer   | 订单id
symbol              | string    | 交易对, 例如btcusdt, ethbtc
price               | string    | limit order的交易价格
created-at          | int       | 订单创建的调整为北京时间的时间戳，单位毫秒
type                | string    | 订单类型
filled-amount       | string    | 订单中已成交部分的数量
filled-cash-amount  | string    | 订单中已成交部分的总价格
filled-fees         | string    | 已交交易手续费总额
source              | string    | 现货交易填写“api”
state               | string    | 订单状态，包括submitted, partical-filled, cancelling

## 批量撤销订单（open orders）

此接口发送批量撤销订单的请求。

<aside class="warning">此接口只提交取消请求，实际取消结果需要通过订单状态，撮合状态等接口来确认。</aside>

### HTTP 请求

- POST ` /v1/order/orders/batchCancelOpenOrders`


### 请求参数

| 参数名称     | 是否必须 | 类型     | 描述           | 默认值  | 取值范围 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true  | string | 账户ID     |     |      |
| symbol     | false | string | 交易对     |      |   单个交易对字符串，缺省将返回所有符合条件尚未成交订单  |
| side | false | string | 主动交易方向 |      |   “buy”或“sell”，缺省将返回所有符合条件尚未成交订单   |
| size | false | int | 所需返回记录数  |  100 |   [0,100]   |


> Response:

```json
{
  "status": "ok",
  "data": {
    "success-count": 2,
    "failed-count": 0,
    "next-id": 5454600
  }
}
```


### 响应数据


| 参数名称 | 是否必须 | 数据类型   | 描述    | 取值范围 |
| ---- | ---- | ------ | ----- | ---- |
| success-count | true | int | 成功取消的订单数 |     |
| failed-count | true | int | 取消失败的订单数 |     |
| next-id | true | long | 下一个符合取消条件的订单号 |    |

## 批量撤销订单

此接口同时为多个订单（基于id）发送取消请求。

### HTTP 请求

- POST ` /v1/order/orders/batchcancel`

```json
{
  "order-ids": [
    "1", "2", "3"
  ]
}
```

### 请求参数

| 参数名称  | 是否必须 | 类型   | 描述   | 默认值  | 取值范围 |
| ---- | ---- | ---- | ----  | ---- | ---- |
| order-ids | true | list | 撤销订单ID列表 |  |单次不超过50个订单id|


> Response:

```json
{  
  "data": {
    "success": [
      "1",
      "3"
    ],
    "failed": [
      {
        "err-msg": "记录无效",
        "order-id": "2",
        "err-code": "base-record-invalid"
      }
    ]
  }
}
```

### 响应数据

| 字段名称 | 数据类型 | 描述
| ---- | ----- | ---- |
| data | map | 撤单结果

## 查询订单详情

此接口返回指定订单的最新状态和详情。

### HTTP 请求

- GET `/v1/order/orders/{order-id}`


### 请求参数

| 参数名称     | 是否必须 | 类型  | 描述   | 默认值  | 取值范围 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 订单ID，填在path中 |      |      |


> Response:

```json
{  
  "data": 
  {
    "id": 59378,
    "symbol": "ethusdt",
    "account-id": 100009,
    "amount": "10.1000000000",
    "price": "100.1000000000",
    "created-at": 1494901162595,
    "type": "buy-limit",
    "field-amount": "10.1000000000",
    "field-cash-amount": "1011.0100000000",
    "field-fees": "0.0202000000",
    "finished-at": 1494901400468,
    "user-id": 1000,
    "source": "api",
    "state": "filled",
    "canceled-at": 0,
    "exchange": "huobi",
    "batch": ""
  }
}
```

### 响应数据

| 字段名称     | 是否必须  | 数据类型   | 描述   | 取值范围     |
| ----------------- | ----- | ------ | -------  | ----  |
| account-id        | true  | long   | 账户 ID    |       |
| amount            | true  | string | 订单数量              |    |
| canceled-at       | false | long   | 订单撤销时间    |     |
| created-at        | true  | long   | 订单创建时间    |   |
| field-amount      | true  | string | 已成交数量    |     |
| field-cash-amount | true  | string | 已成交总金额     |      |
| field-fees        | true  | string | 已成交手续费（买入为币，卖出为钱） |     |
| finished-at       | false | long   | 订单变为终结态的时间，不是成交时间，包含“已撤单”状态    |     |
| id                | true  | long   | 订单ID    |     |
| price             | true  | string | 订单价格       |     |
| source            | true  | string | 订单来源   | api |
| state             | true  | string | 订单状态   | submitting , submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| symbol            | true  | string | 交易对   | btcusdt, ethbtc, rcneth ... |
| type              | true  | string | 订单类型   | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

## 成交明细

此接口返回指定订单的成交明细。

### HTTP 请求

- GET `/v1/order/orders/{order-id}/matchresults`



### 请求参数

| 参数名称  | 是否必须 | 类型  | 描述  | 默认值  | 取值范围 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 订单ID，填在path中 |      |      |


> Response:

```json
{  
  "data": [
    {
      "id": 29553,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "9.1155000000",
      "filled-fees": "0.0182310000",
      "created-at": 1494901400435
    }
    ...
  ]
}
```

### 响应数据

<aside class="notice">返回的主数据对象为一个对象数组，其中每一个元件代表一个交易结果。</aside>

| 字段名称    | 是否必须 | 数据类型   | 描述   | 取值范围     |
| ------------- | ---- | ------ | -------- | -------- |
| created-at    | true | long   | 成交时间     |    |
| filled-amount | true | string | 成交数量     |    |
| filled-fees   | true | string | 成交手续费    |     |
| id            | true | long   | 订单成交记录ID |     |
| match-id      | true | long   | 撮合ID     |     |
| order-id      | true | long   | 订单 ID    |      |
| price         | true | string | 成交价格  |    |
| source        | true | string | 订单来源  | api      |
| symbol        | true | string | 交易对   | btcusdt, ethbtc, rcneth ...  |
| type          | true | string | 订单类型   | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

## 搜索历史订单

此接口基于搜索条件查询历史订单。

### HTTP 请求

- GET `/v1/order/orders`

```json
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
```


### 请求参数

| 参数名称   | 是否必须  | 类型     | 描述   | 默认值  | 取值范围   |
| ---------- | ----- | ------ | ------  | ---- | ----  |
| symbol     | true  | string | 交易对      |      |btcusdt, ethbtc, rcneth ...  |
| types      | false | string | 查询的订单类型组合，使用','分割  |      | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |
| start-date | false | string | 查询开始日期, 日期格式yyyy-mm-dd |      |      |
| end-date   | false | string | 查询结束日期, 日期格式yyyy-mm-dd |      |    |
| states     | true  | string | 查询的订单状态组合，使用','分割  |      | submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| from       | false | string | 查询起始 ID   |      |    |
| direct     | false | string | 查询方向   |      | prev 向前，时间（或 ID）正序；next 向后，时间（或 ID）倒序）    |
| size       | false | string | 查询记录大小      |      |         |


> Response:

```json
{  
  "data": [
    {
      "id": 59378,
      "symbol": "ethusdt",
      "account-id": 100009,
      "amount": "10.1000000000",
      "price": "100.1000000000",
      "created-at": 1494901162595,
      "type": "buy-limit",
      "field-amount": "10.1000000000",
      "field-cash-amount": "1011.0100000000",
      "field-fees": "0.0202000000",
      "finished-at": 1494901400468,
      "user-id": 1000,
      "source": "api",
      "state": "filled",
      "canceled-at": 0,
      "exchange": "huobi",
      "batch": ""
    }
    ...
  ]
}
```

### 响应数据

| 参数名称    | 是否必须  | 数据类型   | 描述   | 取值范围   |
| ----------------- | ----- | ------ | ----------------- | ----  |
| account-id        | true  | long   | 账户 ID    |     |
| amount            | true  | string | 订单数量    |   |
| canceled-at       | false | long   | 接到撤单申请的时间   |    |
| created-at        | true  | long   | 订单创建时间   |    |
| field-amount      | true  | string | 已成交数量   |    |
| field-cash-amount | true  | string | 已成交总金额    |    |
| field-fees        | true  | string | 已成交手续费（买入为基础币，卖出为计价币） |       |
| finished-at       | false | long   | 最后成交时间    |   |
| id                | true  | long   | 订单ID    |    |
| price             | true  | string | 订单价格  |    |
| source            | true  | string | 订单来源   | api  |
| state             | true  | string | 订单状态    | submitting , submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| symbol            | true  | string | 交易对    | btcusdt, ethbtc, rcneth ... |
| type              | true  | string | 订单类型  | submit-cancel：已提交撤单申请  ,buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

## 当前和历史成交

此接口基于搜索条件查询当前和历史成交记录。

### HTTP 请求

- GET `/v1/order/matchresults`


### 请求参数

| 参数名称   | 是否必须  | 类型  | 描述   | 默认值  | 取值范围    |
| ---------- | ----- | ------ | ------ | ---- | ----------- |
| symbol     | true  | string | 交易对   | NA |  btcusdt, ethbtc, rcneth ...  |
| types      | false | string | 查询的订单类型组合，使用','分割   |      | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |
| start-date | false | string | 查询开始日期, 日期格式yyyy-mm-dd | -61 days     | [-61day, today] |
| end-date   | false | string | 查询结束日期, 日期格式yyyy-mm-dd |   today   |  [start-date, today]  |
| from       | false | string | 查询起始 ID    |   订单成交记录ID（最大值）   |     |
| direct     | false | string | 查询方向    |   默认 next， 成交记录 ID 由大到小排序   | prev 向前，时间（或 ID）正序；next 向后，时间（或 ID）倒序）   |
| size       | false | string | 查询记录大小    |   100   | [1，100]  |


> Response:

```json
{  
  "data": [
    {
      "id": 29553,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "9.1155000000",
      "filled-fees": "0.0182310000",
      "created-at": 1494901400435
    }
    ...
  ]
}
```

### 响应数据

<aside class="notice">返回的主数据对象为一个对象数组，其中每一个元件代表一个交易结果。</aside>

| 参数名称   | 是否必须 | 数据类型   | 描述   | 取值范围   |
| ------------- | ---- | ------ | -------- | ------- |
| created-at    | true | long   | 成交时间     |    |
| filled-amount | true | string | 成交数量     |    |
| filled-fees   | true | string | 成交手续费    |    |
| id            | true | long   | 订单成交记录 ID |    |
| match-id      | true | long   | 撮合 ID     |    |
| order-id      | true | long   | 订单 ID    |    |
| price         | true | string | 成交价格     |    |
| source        | true | string | 订单来源     | api   |
| symbol        | true | string | 交易对      | btcusdt, ethbtc, rcneth ...  |
| type          | true | string | 订单类型     | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

# 借贷

<aside class="notice">访问借贷相关的接口需要进行签名认证。</aside>

<aside class="notice">目前杠杆交易仅支持部分以 USDT 和 BTC 为报价币种的交易对。</aside>

## 资产划转

此接口用于现货账户与杠杆账户的资产互转。

从现货账户划转至杠杆账户 `transfer-in`，从杠杆账户划转至现货账户 `transfer-out`

### HTTP 请求

- POST ` /v1/dw/transfer-in/margin`

- POST ` /v1/dw/transfer-out/margin`

```json
{
  "symbol": "ethusdt",
  "currency": "eth",
  "amount": "1.0"
}
```


### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
symbol     | string    | true     | NA      | 交易对, e.g. btcusdt, ethbtc
currency   | string    | true     | NA      | 币种
amount     | string    | true     | NA      | 划转数量


> Response:

```json
{  
  "data": 1000
}
```

### 响应数据


参数名称 | 数据类型 | 描述
------ | ------- | -----
data   | integer | Transfer id


## 申请借贷

此接口用于申请借贷.

### HTTP 请求

- POST ` /v1/margin/orders`

```json
{
  "symbol": "ethusdt",
  "currency": "eth",
  "amount": "1.0"
}
```


### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
symbol     | string    | true     | NA      | 交易对, e.g. btcusdt, ethbtc
currency   | string    | true     | NA      | 币种
amount     | string    | true     | NA      | 借贷数量

> Response:

```json
{  
  "data": 1000
}
```


### 响应数据

字段名称| 数据类型 | 描述
-------| ------  | ----
data   | integer | Margin order id



## 归还借贷

此接口用于归还借贷.

### HTTP 请求

- POST ` /v1/margin/orders/{order-id}/repay`

```json
{
  "amount": "1.0"
}
```


### 请求参数

参数名称 | 数据类型 | 是否必需 | 描述
---------  | --------- | -------- | -----------
order-id   | string    | true     | 借贷订单 ID，写在 url path 中
amount     | string    | true     | 归还币种数量


> Response:

```json
{  
  "data": 1000
}
```

### 响应数据


参数名称     | 数据类型 | 描述
-------  | ------- | -----------
data     | integer | Margin order id


## 查询借贷订单

此接口基于指定搜索条件返回借贷订单。

### HTTP 请求

- GET ` /v1/margin/loan-orders`

### 请求参数

| 参数名称       | 是否必须  | 类型     | 描述    | 默认值  | 取值范围   |
| ----- | ----- | ------ |  -------  | ---- |  ----  |
| symbol | true | string | 交易对  |  |  |
| start-date | false | string | 查询开始日期, 日期格式yyyy-mm-dd  |     |    |
| end-date | false | string | 查询结束日期, 日期格式yyyy-mm-dd  |    |    |
| states | false | string | 状态 |     |   |
| from   | false | string | 查询起始 ID  |    |     |
| direct | false | string | 查询方向     |    | prev 向前，时间（或 ID）正序；next 向后，时间（或 ID）倒序） |
| size   | false | string | 查询记录大小  |    |     |

> Response:

```json
{  
  "data": [
    {
      "loan-balance": "0.100000000000000000",
      "interest-balance": "0.000200000000000000",
      "interest-rate": "0.002000000000000000",
      "loan-amount": "0.100000000000000000",
      "accrued-at": 1511169724531,
      "interest-amount": "0.000200000000000000",
      "symbol": "ethbtc",
      "currency": "btc",
      "id": 394,
      "state": "accrual",
      "account-id": 17747,
      "user-id": 119913,
      "created-at": 1511169724531
    }
  ]
}
```

### 响应数据


| 字段名称 | 是否必须 | 数据类型 | 描述 | 取值范围 |
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |  订单号 | |
|   user-id  |  true  |  long  | 用户ID | |
|   account-id  |  true  |  long  |  账户ID | |
|   symbol  |  true  |  string  |  交易对 | |
|   currency  |  true  |  string  |  币种 | |
| loan-amount | true |string | 借贷本金总额 | |
| loan-balance | true | string | 未还本金 | |
| interest-rate | true | string | 利率 | |
| interest-amount | true | string | 利息总额 | |
| interest-balance | true | string | 未还利息 | |
| created-at | true | long | 借贷发起时间 | |
| accrued-at | true | long | 最近一次计息时间 | |
| state | true | string | 订单状态 |created 未放款，accrual 已放款，cleared 已还清，invalid 异常|


## 借贷账户详情

此接口返回借贷账户详情。

### HTTP 请求

- GET `/v1/margin/accounts/balance`

```json
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
```

### 请求参数

| 参数名称 | 是否必须 | 类型 | 描述 | 默认值 | 取值范围 |
|---------|---------|-----|-----|-------|--------|
| symbol | false | string | 交易对，作为get参数  |  |  |

> Response:

```json
{  
  "data": [
    {
      "id": 18264,
      "type": "margin",
      "state": "working",
      "symbol": "btcusdt",
      "fl-price": "0",
      "fl-type": "safe",
      "risk-rate": "475.952571086994250554",
      "list": [
          {
              "currency": "btc",
              "type": "trade",
              "balance": "1168.533000000000000000"
          },
          {
              "currency": "btc",
              "type": "frozen",
              "balance": "0.000000000000000000"
          },
          {
              "currency": "btc",
              "type": "loan",
              "balance": "-2.433000000000000000"
          },
          {
              "currency": "btc",
              "type": "interest",
              "balance": "-0.000533000000000000"
          },
          {
              "currency": "btc",
              "type": "transfer-out-available",//可转btc
              "balance": "1163.872174670000000000"
          },
          {
              "currency": "btc",
              "type": "loan-available",//可借btc
              "balance": "8161.876538350676000000"
          }
      ]
    }
  ]
}
```

### 响应数据

| 字段名称 | 是否必须 | 数据类型 | 描述 | 取值范围 |
|-----|-----|-----|-----|------|
| symbol  |  true  |  string  |  交易对 | |
| state  |  true  |  string  |  账户状态 | working,fl-sys,fl-mgt,fl-end |
| risk-rate | true | object | 风险率 | |
| fl-price | true | string | 爆仓价 | |
| list | true | array | 子账号列表 | |


# ETF（HB10）

## 基本信息

用户可以通过该接口取得关于 ETF 换入换出的 基本信息，包括一次换入最小量，一次换入最大量，一 次换出最小量，一次换出最大量，换入费率，换出费率，最新 ETF 换入换出状态，以及 ETF 的成分结构。


### HTTP 请求

- GET `/etf/swap/config`

### 请求参数

参数|是否必填|数据类型|长度|说明|取值范围|
-----|-----|-----|------|-------|------|
etf_name| true | string |- | etf基金名称 | hb10|

> Response:

```json
{
  "code": 200,
  "data": {
    "purchase_min_amount": 10000,
    "purchase_max_amount": 100000,
    "redemption_min_amount": 10000,
    "redemption_max_amount": 10000,
    "purchase_fee_rate": 0.001,
    "redemption_fee_rate": 0.002,
    "etf_status":1,
    "unit_price":
    [
      {
        "currency": "eth",
        "amount": 19.9
      },
      {
        "currency": "btc",
        "amount": 9.9
      }
    ]
  },
  "message": null,
  "success": true
}
```

### 响应数据

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
purchase_min_amount | true| int | - | 最小单次换入数量|      |
purchase_max_amount  | False| int | - | 最大单次换入数量 |      |
redemption_min_amount  | true| int | - | 最小单次换出数量 |      |
redemption_max_amount  | False| int | - | 最大单次换出数量 |      |
purchase_fee_rate  | true| double | (5,4)  | 换入费率 |      |
redemption_fee_rate  | true| double | (5,4) | 换出费率 |      |
etf_status  | true| int | - | 换入换出状态 | 状态： 正常 – 1;  由调仓引起的换入换出暂停 - 2; 其他原因引起的换入换出暂停 - 3; 换入暂停 - 4; 换出暂停 – 5  |
unit_price  | true| Array | - | ETF成分信息，包含成分币代码和对应的数量 | 调仓会引起成分信息发生变化  |

- unit_price

参数|是否必填|数据类型|长度|说明|
-----|-----|-----|------|-------|
currency| true | string |- | 成分币币种 |
amount| true | double |- | 成分币数量 |


## 换入换出

用户可以通过该接口取得关于 ETF 换入（swap/in）换出（swap/out）的 基本信息，包括一次换入最小量，一次换入最大量，一次换出最小量，一次换出最大量，换入费率，换出费率，最新 ETF 换入换出状态，以及 ETF 的成分结构。


### HTTP 请求

- POST ` /etf/swap/in `

- POST ` /etf/swap/out`

### 请求参数

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
etf_name  | true| string | - | etf基金名称|    hb10  |
amount  | true| int | - | 换入数量  (POST /etf/swap/in) 或 换出数量 (POST /etf/swap/out) | 换入换出数量的范围请参照接口GET /etf/swap/config 提供的相应范围 |


> Response:

```json
{
    "code": 200,
    "data": null,
    "message": null,
    "success": true
}
```


### 响应数据


参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
code | true| int | - | 结果返回码|   请参照返回码解释表 |
data | true|   | - |  |     |
message | true|   | - |  |     |
success | true| Boolean | - | 请求是否成功|  true or false |

* 返回码解释表

返回码|说明|
--|--|
200|正常|
10404|基金代码不正确或不存在|
13403|账户余额不足|
13404|基金调整中，不能换入换出|
13405|因配置项问题基金不可换入换出|
|13406|非API调用，请求被拒绝
|13410|API签名错误
|13500|系统错误
|13601|调仓期：暂停换入换出
|13603|其他原因，暂停换入和换出
|13604|暂停换入
|13605|暂停换出
|13606|换入或换出的基金份额超过规定范围

## 操作记录

用户可以通过该接口取得关于 ETF 换入换出操 作的明细记录。最多返回 100 条记录。


### HTTP 请求

- GET `/etf/swap/list `

### 请求参数

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
-----------|------------|-----------|------------|----------|--|
etf_name | true| string | - | etf基金名称|   hb10 |
offset | true|  int | - | 开始位置 | >=0. 比如，当offset=0, 开始位置就 是最新的这一条记录。 |
limit | true|  int  | - |最大返回记录条数|  [1, 100]  |

> Response:

```json
{
  "code": 200,
  "data": [
    {
      "id": 112222,
      "gmt_created": 1528855872323,
      "currency": "hb10",
      "amount": 11.5,
      "type": 1,
      "status": 2,
      "detail": 
      {
        "used_ currency_list": 
        [
          {
            "currency": "btc",
            "amount": 0.666
          },
          {
            "currency": "eth",
            "amount": 0.666
          }
        ],
        "rate": 0.002,
        "fee": 100.11,
        "point_card_amount":1.0,
        "obtain_ currency_list": 
        [
          {
            "currency": "hb10",
            "amount": 1000
          }
        ]
      }
    },
    {
      "id": 112223,
      "gmt_created": 1528855872323,
      "currency": "hb10",
      "amount": 11.5,
      "type": 2,
      "status": 1,
      "detail": 
      {
        "used_ currency_list": 
        [
          {
            "currency": "btc",
            "amount": 0.666
          },
          {
            "currency": "eth",
            "amount": 0.666
          }
        ],
        "rate": 0.002,
        "fee": 100.11,
        "point_card_amount":1.0,
        "obtain_ currency_list":
        [
          {
            "currency": "hb10",
            "amount": 1000
          }
        ]
      }
    }
  ],
  "message": null,
  "success": true
}
```

### 响应数据


参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 |
---|------- |------   |---- |-----|--------|
id | true| long | - |操作ID |     |
gmt_created | true| long | - |操作时间（毫秒） |     |
currency | true| string | - |基金名称 |     |
amount | true| double | - |基金数量 |     |
type | true| int | - |操作类型 |    换入-1；换出-2 |
status | true| int | - |操作结果状态 |     成功-2|
detail | true| Detail[] | - |详情 |     |

Detail

参数|是否必填|数据类型|长度|说明|
-----|-----|-----|------|-------|
used_ currency_list| ture| Currency[]| -| 换出的资产列表。如果是换入，该参数包括的是用于换入的成分币详情。如果是换出，该参数则是用于换出的基金详情。|
rate|ture| double| -|费率|
fee|ture| double| -|实际收取的手续费|
point_card_amount| ture| double|-|用点卡折扣的手续费|
obtain_ currency_list| ture| Currency[]| -|换回的资产列表。如果是换入，该参数包括的是用 于换出的基金详情。如果是换出，该参数则是用于 换入的成分币详情。 |

Currency

参数|是否必填|数据类型|长度|说明|
-----|-----|-----|------|-------|
currency| true | string |- | 成分币名称或基金名称 |
amount| true | double |- | 数量 |

# Websocket 订阅

  - <a href='https://github.com/huobiapi/API_Docs/wiki/WS_request'>Websocket 文档 </a>

<br>

# 合约交易API接口

## 接口列表

 |**接口类型**|    **接口数据类型**   |**请求方法**     |                                                                                                                                                   **类型**  | **描述**                    | **需要验签**|
  |----------- |------------------ |------------------------------------------------------------------------------------------------------------------------------------------------------------------- |---------- |---------------------------- |--------------|
  |Restful     |基础信息接口           |api/v1/contract_contract_info |                                                                                                                     GET        |获取合约信息                 |否|
  |Restful     |基础信息接口          |api/v1/contract_index |                                                                                                                             GET        |获取合约指数信息             |否|
  |Restful     |基础信息接口           |api /v1/contract_price_limit|                                                                                                                     GET        |获取合约最高限价和最低限价   |否|
  |Restful     |基础信息接口           | api/v1/contract_open_interest|                                                                                                                      GET        |获取当前可用合约总持仓量     |否|
  |Restful     |基础信息接口           | api/v1/contract_delivery_price|                                                                                                                      GET        |获取预估交割价    |否|
  |Restful     |市场行情接口           | /market/depth |                                                                                                                                    GET        |获取行情深度数据            |否|
  |Restful     |市场行情接口          |/market/history/kline |                                                                                                                            GET        |获取K线数据                  |否|
  |Restful     |市场行情接口          | /market/detail/merged |                                                                                                                         GET        |获取聚合行情                 |否|
  |Restful     |市场行情接口           | /market/trade |                                                                                                                                    GET        |获取市场最近成交记录         |否|
  |Restful     |市场行情接口           |/market/history/trade |                                                                                                                             GET        |批量获取最近的交易记录       |否|
  |Restful     |资产接口           | api/v1/contract_account_info |                                                                                                                   POST       |获取用户账户信息             |是|
  |Restful     |资产接口           |api/v1/contract_position_info |                                                                                                                    POST       |获取用户持仓信息             |是|
  |Restful     |交易接口           |api/v1/contract_order |                                                                                                                            POST       |合约下单                     |是|
  |Restful     |交易接口           |api/v1/contract_batchorder |                                                                                                                        POST       |合约批量下单                 |是|
  |Restful     |交易接口           |api/v1/contract_cancel |                                                                                                                            POST       |撤销订单                     |是|
  |Restful     |交易接口           |api/v1/contract_cancelall |                                                                                                                         POST       |全部撤单                     |是|
  |Restful     |交易接口          |api/v1/contract_order_info |                                                                                                                       POST       |获取合约订单信息             |是|
  |Restful     |交易接口           | api/v1/contract_order_detail |                                                                                                                   POST       |获取订单明细信息             |是|
  |Restful     |交易接口           | api/v1/contract_openorders |                                                                                                                       POST       |获取合约当前未成交委托       |是|
  |Restful     |交易接口           |api/v1/contract_hisorders |                                                                                                                        POST       |获取合约历史委托             |是|
## 访问地址
| 访问地址 | 适用站点 | 适用功能 | 适用交易对 |
|----|----|----|----|
| https://api.hbdm.com| 火币合约|   行情     | 火币合约的交易品种  |
## 签名认证

#### <a name="199">认证方式</a>

用户私有接口(资产接口、交易接口)跟用户相关的所有接口都需加密验签，签证用户合法性

签名认证方式跟pro现货签名一致，详情请参考https://github.com/huobiapi/API_Docs/wiki/REST_authentication

#### <a name="200">访问次数限制</a>

公开行情接口和用户私有接口都有访问次数限制
* 普通用户，需要密钥的私有接口，每个UID 1秒最多10次请求(该UID的所有币种和不同到期日的合约的所有私有接口共享1秒10次的额度)
* 其他非行情类的公开接口，比如获取指数信息，限价信息，交割结算、平台持仓信息等，所有用户都是每个IP一秒最多20次请求（所有该IP的非行情类的公开接口请求共享1秒20次的额度）
* 行情类的公开接口，比如：获取K线数据、获取聚合行情、市场行情、获取市场最近成交记录：

    （1） restful接口：同一个IP,  1秒最多200个请求 

    （2）  websocket：req请求，同一时刻最多请求50次；sub请求，无限制，服务器主动推送数据

* 如果您对API访问频率有特殊需求，请联系 dm_mm@huobi.com，邮件标题“申请提升API限频”，邮件中说明白：

    （1）您的火币UID

    （2）您的应用程序的目的和预期的增长

    （3）您所需的速率限制

  我们工作人员将及时跟进您的限频提升申请。
## 市场行情接口

### 获取合约信息 

URL api/v1/contract_contract_info

**请求参数**

|**参数名称**     |**参数类型**   |**必填**   |**描述**|
|---------------- |-------------- |---------- |------------------------------------------------------------|
  |symbol           |string         |false|      "BTC","ETH"...|
  |contract_type   |string         |false|      合约类型: （this_week:当周 next_week:下周 quarter:季度）|
  |contract_code   |string         |false|      BTC180914|

**备注**： 如果不填，默认查询所有所有合约信息;
如果contract_code填了值，那就按照contract_code去查询;
如果contract_code没有填值，则按照symbol+contract_type去查询;

**返回参数**

  |**参数名称**               |**是否必须**   |**类型**   |**描述**                          |**取值范围**|
  |-------------------------- |-------------- |---------- |--------------------------------- |--------------------------------------------------------------------------------------------------------------------|
  |status                     |true           |string     |请求处理结果                      |"ok" , "error"|
  |\<list\>(属性名称: data)    |                |           |                               | |
  |symbol                     |true           |string     |品种代码                          |"BTC","ETH"...|
  |contract_code             |true           |string     |合约代码                          |"BTC180914" ...|
  |contract_type             |true           |string     |合约类型                          |当周:"this_week", 次周:"next_week", 季度:"quarter"|
  |contract_size             |true           |decimal    |合约面值，即1张合约对应多少美元   |10, 100...|
  |price_tick                |true           |decimal    |合约价格最小变动精度             | 0.001, 0.01...|
  |delivery_date             |true           |string     |合约交割日期                     | 如"20180720"|
  |create_date               |true           |string     |合约上市日期                      |如"20180706"|
  |contract_status           |true           |int        |合约状态                          |合约状态: 0:已下市、1:上市、2:待上市、3:停牌，4:暂停上市中、5:结算中、6:交割中、7:结算完成、8:交割完成、9:暂停上市|
  |\</list\>    |             |               |                     |        |                 
  |ts                         |true           |long       |响应生成时间点，单位：毫秒  |      

**示例**

- GET 
```shell
curl "https://api.hbdm.com/api/v1/contract_contract_info"
```
>Response
```json
    {
      "status": "ok",
      "data": [
        {
          "symbol": "BTC",
          "contract_code": "BTC180914",
          "contract_type": "this_week",
          "contract_size": 100,
          "price_tick": 0.001,
          "delivery_date": "20180704",
          "create_date": "20180604",
          "contract_status": 1
         }
        ],
      "ts":158797866555
    }
```

### 获取合约指数信息

URL  api/v1/contract_index

**请求参数**

  |**参数名称**   |**参数类型**   |**必填**   |**描述**|
  |-------------- |-------------- |---------- |----------------|
  |symbol         |string         |true       |"BTC","ETH"...|

**返回参数**

  |**参数名称**               |**是否必须**   |**类型**   |**描述**                     |**取值范围**|
  |--------------------------| --------------| ----------| ---------------------------- |----------------|
  |status                    | true           |string     |请求处理结果                 |"ok" , "error"|
  |\<list\>(属性名称: data)    |                |           |                           ||
  |symbol                     |true           |string     |指数代码                    | "BTC","ETH"...|
  |index_price               |true           |decimal    |指数价格   |                  |
  |index_ts                |true           |long   |响应生成时间点，单位：毫秒   |                  |
  |\</list\>               |                |           |                           ||                                                            
  |ts                         |true           |long       |时间戳，单位：毫秒   |

**示例**
- GET  
```shell
crul "https://api.hbdm.com/api/v1/contract_index?symbol=BTC"
```
> Response
```json
    {
      "status":"ok",
      "data": [
         {
           "symbol": "BTC",
           "index_price":471.0817,
           "index_ts": 1490759594752
          }
        ],
      "ts": 1490759594752
    }
```

### 获取合约最高限价和最低限价

URL api/v1/contract_price_limit

**请求参数**

  |**参数名称**     |**参数类型**   |**必填**   |**描述**|
  |----------------| --------------| ---------- |-----------------------------------------------------------------|
  |symbol           |string         |false      |"BTC","ETH"...|
  |contract_type   |string         |false      |合约类型 (当周:"this_week", 次周:"next_week", 季度:"quarter")|
  |contract_code   |string         |false      |BTC180914 ...|

**备注**：如果contract_code填了值，那就按照contract_code去查询，如contract_code没有填值，则按照symbol+contract_type去查询，两个查询条件必填一个

**返回参数**

  |**参数名称**               |**是否必须**   |**类型**   |**描述**                     |**取值范围**|
  |-------------------------- |-------------- |---------- |---------------------------- |------------------------------------------------------|
  status|true|string|请求处理结果|"ok" ,"error”|
  \<list\>(属性名称: data)||||
  symbol|true|string|品种代码|"BTC","ETH" ...
  high_limit||true|decimal    最高买价|
  low_limit|| true|decimal    最低卖价|
  contract_code|  true|string|合约代码|如"BTC180914" ...
  contract_type|  true|string|合约类型|当周:"this_week", 次周:"next_week", 季度:"quarter"
  \<list\>||||
  ts|    true|long|  响应生成时间点，单位：毫秒   

**示例**
- GET  
```shell
curl "https://api.hbdm.com/api/v1/contract_price_limit?symbol=BTC&contract_type=this_week"
```
> Response
```json
    {
      "status":"ok",
      "data": 
       [{
          "symbol":"BTC",
          "high_limit":443.07,
          "low_limit":417.09,
          "contract_code":"BTC180914",
          "contract_type":"this_week"
         }],
      "ts": 1490759594752
    }
```

### 获取当前可用合约总持仓量 

URL api/v1/contract_open_interest

**请求参数**

  |**参数名称**|**参数类型**   |**必填**   |**描述**|
  |---------------- |-------------- |---------- |-----------------------------------------------------------------|
 | symbol|string|    false| "BTC","ETH"...|
  |contract_type|   string|    false| 合约类型 (当周:"this_week", 次周:"next_week", 季度:"quarter")|
  |contract_code|   string|    false| BTC180914|

**返回参数**

  |**参数名称**|    **是否必须**   |**类型**   |**描述**|**取值范围**|
 | -------------------------- |-------------- |---------- |----------------------------| ------------------------------------------------------|
  |status|true|string|请求处理结果| "ok" , "error"|
  |\<list\>(属性名称: data)||||
  |symbol|true|string|品种代码|"BTC", "ETH" ...|
  |contract_type|  true|string|合约类型|当周:"this_week", 次周:"next_week", 季度:"quarter"|
  |volume|true|decimal    持仓量(张)||   
  |amount|true|decimal    持仓量(币)||   
  |contract_code|  true|string|合约代码|如"BTC180914" ...|
  |\</list\>|||||
  |ts|    true|long|  响应生成时间点，单位：毫秒   |

**示例**
- GET 
```shell
curl "https://api.hbdm.com/api/v1/contract_open_interest?symbol=BTC&contract_type=this_week"
```
> Response
```json
    {
      "status":"ok",
      "data":
        {
          "symbol":"BTC",
          "contract_type": "this_week",
          "volume":123,
          "amount":106,
          "contract_code": "BTC180914"
         },
      "ts": 1490759594752
    }
```
### 获取预估交割价

URL api/v1/contract_delivery_price

**请求参数**

  |**参数名称**|**参数类型**   |**必填**   |**描述**|
  |---------------- |-------------- |---------- |-----------------------------------------------------------------|
 | symbol|string|    true| "BTC","ETH"...|


**返回参数**

  |**参数名称**|    **是否必须**   |**类型**   |**描述**|**取值范围**|
 | -------------------------- |-------------- |---------- |----------------------------| ------------------------------------------------------|
  |status|true|string|请求处理结果| "ok" , "error"|
  |\<list\>(属性名称: data)||||
  |delivery_price|  true|string|预估交割价| |
  |\</list\>|||||
  |ts|    true|long|  响应生成时间点，单位：毫秒   |

**示例**
- GET  
```shell
curl "https://api.hbdm.com/api/v1/contract_delivery_price?symbol=BTC"
```
> Response
```json
    {
      "status":"ok",
      "data":
        {
          "delivery_price": 3806.4615259197324414715719     
         },
      "ts": 1490759594752
    }
```

### 获取行情深度数据

URL /market/depth

**请求参数**

  |**参数名称**   |**参数类型**   |**必填**   |**描述**|
  |-------------- |-------------- |---------- |--------------------------------------------------------------------------------|
  |symbol|    string|    true|  如"BTC_CW"表示BTC当周合约，"BTC_NW"表示BTC次周合约，"BTC_CQ"表示BTC季度合约|
  |type|string|    true|  (150档数据)  step0, step1, step2, step3, step4, step5（合并深度1-5）；step0时，不合并深度, (20档数据)  step6, step7, step8, step9, step10, step11（合并深度7-11）；step6时，不合并深度|

**返回参数**

| **参数名称** | **是否必须** | **数据类型** | **描述** | **取值范围** |
| -------- | -------- | -------- |--------------------------------------- | -------------- | 
| ch | true |string | 数据所属的 channel，格式： market.period | | 
| status | true |string | 请求处理结果 | "ok" , "error" | 
| asks | true | object |卖盘,[price(挂单价), vol(此价格挂单张数)], 按price升序 | | 
| bids | true| object | 买盘,[price(挂单价), vol(此价格挂单张数)], 按price降序 | | 
| mrid| true| string | 订单ID | | 
|ts | true | number | 响应生成时间点，单位：毫秒 | |

**tick 说明:**

```
    "tick": {
      "id": 消息id.
      "ts": 消息生成时间，单位：毫秒.
      "bids": 买盘,[price(挂单价), vol(此价格挂单张数)], //按price降序.
      "asks": 卖盘,[price(挂单价), vol(此价格挂单张数)]  //按price升序.
      "ch": 数据所属的 channel,
      "mrid": 订单ID,
      "ts": 时间戳,
      "version": 版本
    }
```
**示例**
- GET 
```shell
curl "https://api.hbdm.com/market/depth?symbol=BTC_CQ&type=step5"
```
> Response
```json
    {
      "ch":"market.BTC_CQ.depth.step5",
      "status":"ok",
        "tick":{
          "asks":[
            [6580,3000],
            [70000,100]
            ],
          "bids":[
            [10,3],
            [2,1]
            ],
          "ch":"market.BTC_CQ.depth.step5",
          "id":1536980854,
          "mrid":6903717,
          "ts":1536980854171,
          "version":1536980854
        },
      "ts":1536980854585
    }
```

### 获取K线数据

URL /market/history/kline

**请求参数**

  |**参数名称**   |**是否必须**  | **类型**   |**描述**   |**默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |---------- |------------ |--------------------------------------------------------------------------------|
  |symbol|    true|string|合约名称||如"BTC_CW"表示BTC当周合约，"BTC_NW"表示BTC次周合约，"BTC_CQ"表示BTC季度合约|
  |period|    true|string|K线类型|| 1min, 5min, 15min, 30min, 60min,4hour,1day, 1mon|
  |size|true|integer    |获取数量   |150|[1,2000]|

**返回参数**

  |**参数名称**   |**是否必须**   |**数据类型**   |**描述**|   **取值范围**|
 | -------------- |-------------- |-------------- |------------------------------------------ |----------------|
  |ch|  true|string|    数据所属的 channel，格式： market.period   |
  |data|true|object|    KLine 数据|| 
  |status|    true|string|    请求处理结果|"ok" , "error"|
  |ts|  true|number|    响应生成时间点，单位：毫秒|| 

**Data说明：**
```
"data": [
  {
    "id": K线id,
    "vol": 成交量(张),
    "count": 成交笔数,
    "open": 开盘价,
    "close": 收盘价,当K线为最晚的一根时，是最新成交价
    "low": 最低价,
    "high": 最高价,
    "amount": 成交量(币), 即 sum(每一笔成交量(张)*单张合约面值/该笔成交价)
   }
]
```

**示例**
- GET  
```shell
curl "https://api.hbdm.com/market/history/kline?period=1min&size=200&symbol=BTC_CQ"
```
> Response
```json
    {
      "ch": "market.BTC_CQ.kline.1min",
      "data": [
        {
          "vol": 2446,
          "close": 5000,
          "count": 2446,
          "high": 5000,
          "id": 1529898120,
          "low": 5000,
          "open": 5000,
          "amount": 48.92
         },
        {
          "vol": 0,
          "close": 5000,
          "count": 0,
          "high": 5000,
          "id": 1529898780,
          "low": 5000,
          "open": 5000,
          "amount": 0
         }
       ],
      "status": "ok",
      "ts": 1529908345313
    }
```

### 获取聚合行情

URL /market/detail/merged

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**   |**默认值**   |**取值范围**|
  |--------------| --------------| ---------- |----------| ------------ |--------------------------------------------------------------------------------|
  |symbol|    true|string|合约名称|如"BTC_CW"表示BTC当周合约，"BTC_NW"表示BTC次周合约，"BTC_CQ"表示BTC季度合约|

**返回参数**

  |**参数名称**   |**是否必须**  | **数据类型**   |**描述**|    **取值范围**|
 | -------------- |-------------- |-------------- |----------------------------------------------------------| ----------------|
  |ch|  true|string|    数据所属的 channel，格式： market.\$symbol.detail.merged   |
  |status|    true|string|    请求处理结果|"ok" , "error"|
  |tick|true|object|    K线数据||
  |ts|  true|number|    响应生成时间点，单位：毫秒|| 

**tick说明:**
```
    "tick": {
      "id": K线id,
      "vol": 成交量（张）,
      "count": 成交笔数,
      "open": 开盘价,
      "close": 收盘价,当K线为最晚的一根时，是最新成交价
      "low": 最低价,
      "high": 最高价,
      "amount": 成交量(币), 即 sum(每一笔成交量(张)*单张合约面值/该笔成交价)
      "bid": [买1价,买1量(张)],
      "ask": [卖1价,卖1量(张)]
     }
```

**示例**
- GET  
```shell
curl "https://api.hbdm.com/market/detail/merged?symbol=BTC_CQ"
```
> Response
```json
    {
      "ch": "market.BTC_CQ.detail.merged",
      "status": "ok",
      "tick": 
        {
          "vol":"13305",
          "ask": [5001, 2],
          "bid": [5000, 1],
          "close": "5000",
          "count": "13305",
          "high": "5000",
          "id": 1529387841,
          "low": "5000",
          "open": "5000",
          "ts": 1529387842137,
          "amount": "266.1"
         },
      "ts": 1529387842137
    }
```

### 获取市场最近成交记录

URL /market/trade

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**   |**默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |---------- |------------ |--------------------------------------------------------------------------------|
  |symbol|    true|string|合约名称|如"BTC_CW"表示BTC当周合约，"BTC_NW"表示BTC次周合约，"BTC_CQ"表示BTC季度合约|

**返回参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**|  **默认值**   |**取值范围**|
  |--------------| --------------| ----------| ---------------------------------------------------------| ------------ |--------------|
  |ch|  true|string|数据所属的 channel，格式： market.\$symbol.trade.detail||
  |status|    true|string|||  "ok","error"
  |tick|true|object|Trade 数据|||   
  |ts|  true|number|发送时间|||

**Tick说明：**
```
    "tick": {
      "id": 消息id,
      "ts": 最新成交时间,
      "data": [
        {
       "id": 成交id,
        "price": 成交价钱,
         "amount": 成交量(张),
         "direction": 主动成交方向,
         "ts": 成交时间
        }
      ]
    }
```

**示例**
- GET  
```shell
curl "https://api.hbdm.com/market/trade?symbol=BTC_CQ"
```
> Response
```json
    {
      "ch": "market.BTC_CQ.trade.detail",
      "status": "ok",
      "tick": {
        "data": [
          {
            "amount": "1",
            "direction": "sell",
            "id": 6010881529486944176,
            "price": "5000",
            "ts": 1529386945343
           }
         ],
        "id": 1529388202797,
        "ts": 1529388202797
        },
      "ts": 1529388202797
    }
```

### 批量获取最近的交易记录

URL /market/history/trade

**请求参数：**

  |**参数名称**   |**是否必须**   |**数据类型**   |**描述**|  **默认值**   |**取值范围**|
  |-------------- |-------------- |-------------- |-------------------- |------------ |--------------------------------------------------------------------------------|
 | symbol|    true|string|    合约名称||如"BTC_CW"表示BTC当周合约，"BTC_NW"表示BTC次周合约，"BTC_CQ"表示BTC季度合约
  |size|false|number|    获取交易记录的数量  | 1| [1, 2000]

**返回参数**

  |**参数名称**   |**是否必须**   |**数据类型**   |**描述**|   **取值范围**|
  |--------------| --------------| --------------| ---------------------------------------------------------| ---------------|
  |ch|  true|string|    数据所属的 channel，格式： market.\$symbol.trade.detail   ||
  |data|true|object|    Trade 数据||
  |status|    true|string||    "ok"，"error"
  |ts|  true|number|    响应生成时间点，单位：毫秒||

**data说明：**
```
    "data": {
      "id": 消息id,
      "ts": 最新成交时间,
      "data": [
        {
          "id": 成交id,
          "price": 成交价,
          "amount": 成交量(张),
          "direction": 主动成交方向,
          "ts": 成交时间
        }
      ]
    }
```

**示例**
- GET  
```shell
curl "https://api.hbdm.com/market/history/trade?symbol=BTC_CQ&size=100"
```
> Response
```json
    {
      "ch": "market.BTC_CQ.trade.detail",
      "status": "ok",
      "ts": 1529388050915,
      "data": [
        {
          "id": 601088,
          "ts": 1529386945343,
          "data": [
            {
             "amount": 1,
             "direction": "sell",
             "id": 6010881529486944176,
             "price": 5000,
             "ts": 1529386945343
             }
           ]
        }
       ]
    }
```


## 资产接口

### 获取用户账户信息

URL  api/v1/contract_account_info

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**   |**默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |----------| ------------ |------------------------------------------|
  |symbol|    false|string|品种代码||"BTC","ETH"...如果缺省，默认返回所有品种|

**返回参数**

  |**参数名称**|    **是否必须**   |**类型**   |**描述**|   **取值范围**|
  |-------------------------- |-------------- |---------- |------------------------------------------ |----------------|
  |status|true|string|请求处理结果|"ok" , "error"|
  |\<list\>(属性名称: data)||||    
  |symbol|true|string|品种代码|"BTC","ETH"...|
  |margin_balance| true|decimal    |账户权益||   
  |margin_position|true|decimal    |持仓保证金（当前持有仓位所占用的保证金）   ||
  |margin_frozen|  true|decimal    |冻结保证金|| 
  |margin_available|true|decimal   |可用保证金|| 
  |profit_real|    true|decimal    |已实现盈亏|| 
  |profit_unreal|  true|decimal    |未实现盈亏|| 
  |risk_rate|| true|decimal    |保证金率||   
  |liquidation_price|    true|decimal    |预估强平价|| 
  |withdraw_available|   true|decimal    |可划转数量|| 
  |lever_rate||true|decimal    |杠杠倍数||   
  |\</list\>||||   
  |ts|    number|    long|  响应生成时间点，单位：毫秒|| 

**示例**
- POST  `"https://api.hbdm.com/api/v1/contract_account_info"    `
> Response
```json
    {
      "status": "ok",
      "data": [
        {
          "symbol": "BTC",
          "margin_balance": 1,
          "margin_position": 0,
          "margin_frozen": 3.33,
          "margin_available": 0.34,
          "profit_real": 3.45,
          "profit_unreal": 7.45,
          "withdraw_available":4.0989898,
          "risk_rate": 100,
          "liquidation_price": 100
         },
        {
          "symbol": "ETH",
          "margin_balance": 1,
          "margin_position": 0,
          "margin_frozen": 3.33,
          "margin_available": 0.34,
          "profit_real": 3.45,
          "profit_unreal": 7.45,
          "withdraw_available":4.7389859,
          "risk_rate": 100,
          "liquidation_price": 100
         }
       ],
      "ts":158797866555
    }
```


### 获取用户持仓信息

URL api/v1/contract_position_info

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**   |**默认值**   |**取值范围**|
  |-------------- |--------------| ---------- |----------| ------------ |------------------------------------------|
  |symbol|    false|string|品种代码||"BTC","ETH"...如果缺省，默认返回所有品种|

**返回参数**

  |**参数名称**|    **是否必须**  | **类型**  | **描述**|**取值范围**|
  |-------------------------- |-------------- |---------- |---------------------------- |------------------------------------------------------|
  |status|true|string|请求处理结果| "ok" , "error"|
  |\<list\>(属性名称: data)||||
  |symbol|true|string|品种代码|"BTC","ETH"...|
  |contract_code|  true|string|合约代码|"BTC180914" ...|
  |contract_type|  true|string|合约类型|当周:"this_week", 次周:"next_week", 季度:"quarter"|
  |volume|true|decimal    |持仓量|  |
  |available| true|decimal    |可平仓数量||   
  |frozen|true|decimal    |冻结数量||
  |cost_open|true|decimal    |开仓均价||
  |cost_hold| true|decimal    |持仓均价||
  |profit_unreal|  true|decimal    |未实现盈亏||   
  |profit_rate|    true|decimal    |收益率| | 
  |profit|true|decimal    收益|    |
  |position_margin|true|decimal    |持仓保证金||   
  |lever_rate|true|int|   杠杠倍数||
  |direction||  true|string|"buy":买 "sell":卖||
  |\</list\>|||||
  |ts|    true|long|  响应生成时间点，单位：毫秒   ||

**示例**
- POST  `https://api.hbdm.com/api/v1/contract_position_info`

> Response
```json
    {
      "status": "ok",
      "data": [
        {
          "symbol": "BTC",
          "contract_code": "BTC180914",
          "contract_type": "this_week",
          "volume": 1,
          "available": 0,
          "frozen": 0.3,
          "cost_open": 422.78,
          "cost_hold": 422.78,
          "profit_unreal": 0.00007096,
          "profit_rate": 0.07,
          "profit": 0.97,
          "position_margin": 3.4,
          "lever_rate": 10,
          "direction":"buy"
         }
        ],
     "ts": 158797866555
    }
```


## 交易接口


### 合约下单 

URL api/v1/contract_order

**请求参数**

  |**参数名**|**参数类型**   |**必填**   |**描述**|
  |-------------------- |-------------- |----------| ---------------------------------------------------------------|
  |symbol|    string|    true| "BTC","ETH"...|
  |contract_type|  string|    true| 合约类型 ("this_week":当周 "next_week":下周 "quarter":季度)|
  |contract_code|  string|    true| BTC180914|
  |client_order_id |   long|false| 客户自己填写和维护，这次一定要大于上一次|
  |price|decimal|   true|  价格|
  |volume|    long|true|  委托数量(张)|
  |direction| string|    true|  "buy":买 "sell":卖|
  |offset|    string|    true|  "open":开 "close":平|
  |lever_rate|int| true|  杠杆倍数[“开仓”若有10倍多单，就不能再下20倍多单]|
  |order_price_type |  string|    true|  订单报价类型 "limit":限价 "opponent":对手价|

**备注**：如果contract_code填了值，那就按照contract_code去下单，如果contract_code没有填值，则按照symbol+contract_type去下单。

**返回参数**

  |**参数名称**|   **是否必须**   |**类型**   |**描述**|**取值范围**|
  |------------------- |-------------- |---------- |-------------------------------------------- |----------------|
  |status|   true|string|请求处理结果|"ok" , "error"|
  |order_id|true|long|  订单ID|| 
  |client_order_id  | true|long|  用户下单时填写的客户端订单ID，没填则不返回  | 
  |ts|  true|long|  响应生成时间点，单位：毫秒||   

**示例**
- POST  `https://api.hbdm.com/api/v1/contract_order`
> Response
```json

    {
      "status": "ok",
      "data": {
		"order_id": 88
	      },
      "ts": 158797866555
    }
```

### 合约批量下单 

URL api/v1/contract_batchorder

**请求参数**

  |**参数名**|    **参数类型**   |**必填**   |**描述**|
  |---------------------------------- |-------------- |---------- |--------------------------------------------------------------|
  |\<list\>(属性名称: orders_data)|| ||  
  |symbol|   string|    false| "BTC","ETH"...|
  |contract_type|string|    false| 合约类型: "this_week":当周 "next_week":下周 "quarter":季度|
  |contract_code|string|    false| BTC180914|
  |client_order_id|  long|false|  客户自己填写和维护，这次一定要大于上一次|
  |price|  decimal|   true|  价格|
  |volume|  long|true|  委托数量(张)|
  |direction|string|    true|  "buy":买 "sell":卖|
  |offset|  string|    true|  "open":开 "close":平|
  |lever_rate|   int| true|  杠杆倍数[“开仓”若有10倍多单，就不能再下20倍多单]|
  |order_price_type| string|    true|  订单报价类型 "limit":限价 "opponent":对手价|
  |\</list\>||||

**备注**：如果contract_code填了值，那就按照contract_code去下单，如果contract_code没有填值，则按照symbol+contract_type去下单。

**返回参数**

  |**参数名称**|  **是否必须**   |**类型**   |**描述**|**取值范围**|
  |----------------------------- |-------------- |---------- |--------------------------------------------| ----------------|
  |status|   true|string|请求处理结果| "ok" , "error"|
  |\<list\>(属性名称: errors)|||| 
  |index|    true|int|   订单索引||
  |err_code|true|int|   错误码||
  |err_msg| true|string|错误信息||
  |\</list\>||||  
  |\<list\>(属性名称: success)||||
  |index|    true|int|   订单索引||
  |order_id|true|long|  订单ID|| 
  |client_order_id|  true|long|  用户下单时填写的客户端订单ID，没填则不返回  | 
  |\</list\>||||
  |ts|  true|long|  响应生成时间点，单位：毫秒|

**示例**
- POST  
`https://api.hbdm.com/api/v1/contract_batchorder`
> Response
```json
    {
      "status": "ok",
      "data": {
        "errors":[
          {
            "index":0,
            "err_code": 200417,
            "err_msg": "invalid symbol"
           },
          {
            "index":3,
            "err_code": 200415,
            "err_msg": "invalid symbol"
           }
         ],
        "success":[
          {
            "index":1,
            "order_id":161256,
            "client_order_id":1344567
           },
          {
            "index":2,
            "order_id":161257,
            "client_order_id":1344569
           }
         ]
       },
      "ts": 1490759594752
    }
```

### 撤销订单 

URL api/v1/contract_cancel

**请求参数**

  |**参数名称**|   **是否必须**   |**类型**   |**描述**|
  |------------------- |-------------- |---------- |--------------------------------------------------------------|
  |order_id|false|string|订单ID（ 多个订单ID中间以","分隔,一次最多允许撤消50个订单 ）|
  |client_order_id|  false|string|客户订单ID(多个订单ID中间以","分隔,一次最多允许撤消50个订单)|
  |symbol|   true|string|"BTC","ETH"...|

**备注**：
order_id和client_order_id都可以用来撤单，同时只可以设置其中一种，如果设置了两种，默认以order_id来撤单。

**返回参数**

  |**参数名称**| **是否必须**  | **类型**  | **描述**| **取值范围**|
  |---------------------------- |-------------- |---------- |--------------------------------------------------| ----------------|
  |status|  true|string|请求处理结果| "ok" , "error"|
  |\<list\>(属性名称: errors)||||
  |order_id|    true|string|订单ID||   
  |err_code|   true|int|   错误码||   
  |err_msg|true|string|错误信息|| 
  |\</list\>||||
  |successes|   true|string|撤销成功的订单的order_id或client_order_id列表  | |
  |ts|true|long|  响应生成时间点，单位：毫秒| |

**示例**
- POST `https://api.hbdm.com/api/v1/contract_cancel`
> Response
```json

    {
  "status": "ok",
  "data": {
    "errors":[
      {
        "order_id":"161251",
        "err_code": 200417,
        "err_msg": "invalid symbol"
       },
      {
        "order_id":161253,
        "err_code": 200415,
        "err_msg": "invalid symbol"
       }
      ],
    "successes":[161256,1344567]
   },
  "ts": 1490759594752
}   
```

### 全部撤单 

URL api/v1/contract_cancelall

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**|
  |-------------- |-------------- |---------- |----------------------------|
  |symbol|    true|string|品种代码，如"BTC","ETH"...|

**返回参数**

  |**参数名称**| **是否必须**   |**类型**  | **描述**|**取值范围**|
  |---------------------------- |-------------- |---------- |---------------------------- |----------------|
  |status|  true|string|请求处理结果| "ok" , "error"| 
  |\<list\>(属性名称: errors)||||
  |order_id|    true|String|订单id| | 
  |err_code|    true|int|   订单失败错误码| |   
  |err_msg|true|int|   订单失败信息|| 
  |\</list\>|||| 
  |successes|    true|string|成功的订单||   
  |ts| true|long|  响应生成时间点，单位：毫秒  || 

**示例**
    POST  https://api.hbdm.com.com/api/v1/contract_cancelall
> Response
    
```json
    多笔订单返回结果(成功订单ID,失败订单ID)
    {
      "status": "ok",
      "data": {
        "errors":[
          {
            "order_id":"161251",
            "err_code": 200417,
            "err_msg": "invalid symbol"
           },
          {
            "order_id":161253,
            "err_code": 200415,
            "err_msg": "invalid symbol"
           }
          ],
        "successes":[161256,1344567]
       },
      "ts": 1490759594752
    }
```


### 获取合约订单信息

URL api/v1/contract_order_info

**请求参数**

  |**参数名称**|   **是否必须**   |**类型**   |**描述**|
  |------------------- |--------------| ---------- |-------------------------------------------------------------|
  |order_id|false|string|订单ID（ 多个订单ID中间以","分隔,一次最多允许查询20个订单 ）|
  |client_order_id   |false|string|客户订单ID(多个订单ID中间以","分隔,一次最多允许查询20个订单)|
  |symbol|   true|string|"BTC","ETH"...|

**备注**：order_id和client_order_id都可以用来查询，同时只可以设置其中一种，如果设置了两种，默认以order_id来查询。

**返回数据**

  |**参数名称**|    **是否必须**   |**类型**   |**描述**| **取值范围**|
  |-------------------------- |-------------- |---------- |-------------------------------------------------------------------------------------------- |----------------------------------------------------|
  |status|true|string|请求处理结果|  "ok" , "error"|
  |\<list\>(属性名称: data)||||| 
  |symbol|true|string|品种代码||| 
  |contract_type|  true|string|合约类型|当周:"this_week", 周:"next_week", 季度:"quarter"|
  |contract_code|  true|string|合约代码| "BTC180914" ...|
  |volume|true|decimal    |委托数量|| 
  |price| true|decimal    |委托价格|| 
  |order_price_type|    true|string|订单报价类型 "limit":限价 "opponent":对手价||   
  |direction|  true|string|"buy":买 "sell":卖||
  |offset|true|string|"open":开 "close":平||
  |lever_rate|true|int|   杠杆倍数|1\\5\\10\\20|
  |order_id|  true|long|  订单ID|| 
  |client_order_id|true|long|  客户订单ID||  
  |created_at|true|long|  成交时间||
  |trade_volume|   true|decimal|    成交数量||
  |trade_turnover| true|decimal    成交总金额||    
  |fee|   true|decimal|    手续费||   
  |trade_avg_price|true|decimal|    成交均价|| 
  |margin_frozen|  true|decimal|    冻结保证金||   
  |profit|true|decimal|    收益||
  |status|true|int|   订单状态|(1准备提交 2准备提交 3已提交 4部分成交 5部分成交已撤单 6全部成交 7已撤单 11撤单中) |  
  |order_type|true|string|   订单类型|  1:报单 、 2:撤单 、 3:强平、4:交割
  |order_source|   true|string|订单来源|（1:system、2:web、3:api、4:m 5:risk、6:settlement） |   
  |\</list\>|||||
  |ts|    true|long|  时间戳||   

**示例**
- POST  `https://api.hbdm.com.com/api/v1/contract_order_info`
> Response
```json
    {
      "status": "ok",
      "data":[
        {
          "symbol": "BTC",
          "contract_type": "this_week",
          "contract_code": "BTC180914",
          "volume": 111,
          "price": 1111,
          "order_price_type": "limit",
          "direction": "buy",
          "offset": "open",
          "lever_rate": 10,
          "order_id": 106837,
          "client_order_id": 10683,
          "order_source": "web",
          "order_type": "1",
          "created_at": 1408076414000,
          "trade_volume": 1,
          "trade_turnover": 1200,
          "fee": 0,
          "trade_avg_price": 10,
          "margin_frozen": 10,
          "profit ": 10,
          "status": 0
         },
        {
          "symbol": "ETH",
          "contract_type": "this_week",
          "contract_code": "ETH180921",
          "volume": 111,
          "price": 1111,
          "order_price_type": "limit",
          "direction": "buy",
          "offset": "open",
          "lever_rate": 10,
          "order_id": 106837,
          "client_order_id": 10683,
          "order_source": "web",
           "order_type": "1",
          "created_at": 1408076414000,
          "trade_volume": 1,
          "trade_turnover": 1200,
          "fee": 0,
          "trade_avg_price": 10,
          "margin_frozen": 10,
          "profit ": 10,
          "status": 0
         }
        ],
      "ts": 1490759594752
    }
```

### 获取订单明细信息

URL api/v1/contract_order_detail

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**|
  |-------------- |-------------- |---------- |------------------------|
  |symbol|    true|string|"BTC","ETH"...|
  |order_id| true|long|  订单id|
  |created_at|  true|long|  下单时间戳|
  |order_type|  true|int|  订单类型，1:报单 、 2:撤单 、 3:强平、4:交割|
  |page_index|    false|int|   第几页,不填第一页|
  |page_size|false|int|   不填默认20，不得多于50|

**返回数据**

  |**参数名称**|  **是否必须**   |**类型**   |**描述**| **取值范围**|
  |----------------------------- |-------------- |---------- |--------------------------------------------- |------------------------------------------------------|
  |status|   true|string|请求处理结果| "ok" , "error"|
  |\<object\> (属性名称: data)|||| 
  |symbol|   true|string|品种代码|| 
  |contract_type|true|string|合约类型|当周:"this_week", 次周:"next_week", 季度:"quarter"|
  |contract_code|true|string|合约代码|"BTC180914" ...|
  |lever_rate|   true|int|   杠杆倍数| 1\5\10\20|
  |direction|true|string|"buy":买 "sell":卖||  
  |offset|   true|string|"open":开 "close":平||
  |volume|   true|decimal    |委托数量|| 
  |price|    true|decimal    |委托价格|| 
  |created_at|   true|long|  成交时间||
  |order_source| true|string|订单来源|| 
  |order_price_type| true|string|订单报价类型 |1限价单 3对手价   |  
  |margin_frozen|true|decimal    |冻结保证金||    
  |profit|   true|decimal    |收益||
  |total_page|   true|int|   总共页数|||
  |current_page| true|int|   当前页数|| 
  |total_size|   true|int|   总条数||   
  |\<list\> (属性名称: trades)|||| 
  |trade_id|true|long|  撮合结果id||    
  |trade_price|  true|decimal    撮合价格||
  |trade_volume| true|decimal    成交量||  
  |trade_turnover|    true|decimal    成交金额|| 
  |trade_fee|   true|decimal    成交手续费||    
  |created_at|   true|long|  创建时间||| 
  |\</list\>||||   
  |\</object \>||||
  |ts|  true|long|  时间戳||

**示例**
- POST  `https://api.hbdm.com/api/v1/contract_order_detail`
> Response
```json
    {
      "status": "ok",
      "data":{
        "symbol": "BTC",
        "contract_type": "this_week",
        "contract_code": "BTC180914",
        "volume": 111,
        "price": 1111,
        "order_price_type": "limit",
        "direction": "buy",
        "offset": "open",
        "status": 1,
        "lever_rate": 10,
        "trade_avg_price": 10,
        "margin_frozen": 10,
        "profit": 10,
        "order_id": 106837,
        "order_source": "web",
        "created_at": 1408076414000,
        "trades":[
          {
            "trade_id":112,
            "trade_volume":1,
            "trade_price":123.4555,
            "trade_fee":0.234,
            "trade_turnover":34.123,
            "created_at": 1490759594752
           }
          ],
        "total_page":15,
        "total_size":3,
        "current_page":3
        },
      "ts": 1490759594752
    }
```
**错误**
```json
    {
     "status":"error",
     "err_code":20029,
     "err_msg": "invalid symbol",
     "ts": 1490759594752
    }
```

### 获取合约当前未成交委托 

URL  /v1/contract_openorders

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**| **默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |------------------------ |------------ |----------------|
  |symbol|    false|string|品种代码|    |"BTC","ETH"...|
  |page_index   | false|int|   ||页码，不填默认第1页| 1| 
  |page_size|false|int|   ||不填默认20，不得多于50 |

**返回参数**

  |**参数名称**|    **是否必须**   |**类型**   |**描述**|   **取值范围**|
  |-------------------------- |-------------- |---------- |--------------------------------------------------------------- |------------------------------------------------------|
  |status|true|string|请求处理结果||
  |\<list\>(属性名称: data)||||   
  |symbol|true|string|品种代码||  
  |contract_type|  true|string|合约类型|当周:"this_week", 次周:"next_week", 季度:"quarter"|
  |contract_code|  true|string|合约代码|"BTC180914" ...|
  |volume|true|decimal    |委托数量||
  |price| true|decimal    |委托价格||   
  |order_price_type|    true|string|订单报价类型 "limit":限价 "opponent":对手价|
  |direction|  true|string|"buy":买 "sell":卖||   
  |offset|true|string|"open":开 "close":平||  
  |lever_rate||true|int|   杠杆倍数|   1\\5\\10\\20|
  |order_id||  true|long|  订单ID||
  |client_order_id|true|long|  客户订单ID||
  |created_at|true|long|  订单创建时间||
  |trade_volume|   true|decimal    |成交数量||  
  |trade_turnover| true|decimal    |成交总金额|| 
  |fee|   true|decimal    |手续费||
  |trade_avg_price|true|decimal    |成交均价||  
  |margin_frozen|  true|decimal    |冻结保证金|| 
  |profit|true|decimal   | 收益||  
  |status|true|int|   订单状态|(3未成交 4部分成交 5部分成交已撤单 6全部成交 7已撤单) |  
  |order_source|   true|string|订单来源||
  |\</list\>||||
  |total_page|true|int|   总页数||
  |current_page|   true|int|   当前页||
  |total_size|true|int|   总条数||
  |ts|    true|long|  时间戳||

**示例**
- POST `https://www.hbdm.com/api/v1/contract_openorders`
> Response
```json
    {
      "status": "ok",
      "data":{
        "orders":[
          {
             "symbol": "BTC",
             "contract_type": "this_week",
             "contract_code": "BTC180914",
             "volume": 111,
             "price": 1111,
             "order_price_type": "limit",
             "direction": "buy",
             "offset": "open",
             "lever_rate": 10,
             "order_id": 106837,
             "client_order_id": 10683,
             "order_source": "web",
             "created_at": 1408076414000,
             "trade_volume": 1,
             "trade_turnover": 1200,
             "fee": 0,
             "trade_avg_price": 10,
             "margin_frozen": 10,
             "status": 1
            }
           ],
        "total_page":15,
        "current_page":3,
        "total_size":3
       },
      "ts": 1490759594752
    }
```


### 获取合约历史委托

URL api/v1/contract_hisorders

**请求参数**

  |**参数名称**   |**是否必须**   |**类型**   |**描述**| **默认值**   |**取值范围**|
  |--------------| --------------| ---------- |------------------------| ------------| ------------------------------------------------------------------------------------------------------|
  |symbol|    true|string|品种代码|   "BTC","ETH"...|
  |trade_type |   true|int|   交易类型|    0:全部,1:买入开多,2: 卖出开空,3: 买入平空,4: 卖出平多,5: 卖出强平,6: 买入强平,7:交割平多,8: 交割平空|
  |type|true|int|   类型|1:所有订单,2:结束状态的订单|
  |status|    true|int|   订单状态|    0:全部,3:未成交, 4: 部分成交,5: 部分成交已撤单,6: 全部成交,7:已撤单|
  |create_date |  true|int|   日期|  7，90（7天或者90天）|
  |page_index  |  false|int|   |页码，不填默认第1页| 1| 
  |page_size|false|int|   |不填默认20，不得多于50   20|

**返回参数**

  |**参数名称**| **是否必须**   |**类型**   |**描述**|**取值范围**|
  |---------------------------- |-------------- |---------- |--------------------------------------------- |------------------------------------------------------|
  |status|  true|string|请求处理结果||  
  |\<object\>(属性名称: data)|||| 
  |\<list\>(属性名称: orders)|||| 
  |order_id|    true|long|  订单ID|  
  |symbol|  true|string|品种代码|
  |contract_type|    true|string|合约类型| 当周:"this_week", 次周:"next_week", 季度:"quarter"|
  |contract_code|    true|string|合约代码| "BTC180914" ...|
  |lever_rate|  true|int|   杠杆倍数| 1\\5\\10\\20|
  |direction|    true|string|"buy":买 "sell":卖||  
  |offset|  true|string|"open":开 "close":平||
  |volume|  true|decimal    |委托数量||
  |price|   true|decimal    |委托价格|| 
  |create_date| true|long|  创建时间|| 
  |order_source|true|string|订单来源|| 
  |order_price_type|true|string|订单报价类型 |"limit":限价 "opponent":对手价 |  
  |margin_frozen|    true|decimal    |冻结保证金||    
  |profit|  true|decimal    |收益||
  |trade_volume|true|decimal    |成交数量|| 
  |trade_turnover|   true|decimal    |成交总金额||    
  |fee||true|decimal    |手续费||   
  |trade_avg_price| true|decimal    |成交均价|| 
  |status|  true|int|   订单状态|| 
  |order_type|  true|int|   订单类型|1:报单 、 2:撤单 、 3:强平、4:交割|
  |\</list\>||||  
  |\</object\>||||
  |total_page|  true|int|   总页数||   
  |current_page|true|int|   当前页||   
  |total_size|  true|int|   总条数||  
  |ts|true|long|  时间戳||  

**示例**
- POST `https://api.hbdm.com/api/v1/contract_hisorders`
> Response
```json
    {
      "status": "ok",
      "data":{
        "orders":[
          {
            "symbol": "BTC",
            "contract_type": "this_week",
            "contract_code": "BTC180914",
            "volume": 111,
            "price": 1111,
            "order_price_type": "limit",
            "direction": "buy",
            "offset": "open",
            "lever_rate": 10,
            "order_id": 106837,
            "client_order_id": 10683,
            "order_source": "web",
            "created_at": 1408076414000,
            "trade_volume": 1,
            "trade_turnover": 1200,
            "fee": 0,
            "trade_avg_price": 10,
            "margin_frozen": 10,
            "profit": 10,
            "status": 1
          }
         ],
        "total_page":15,
        "current_page":3,
        "total_size":3
        },
      "ts": 1490759594752
    }
```

<br>
<br>
<br>
<br>
<br>
