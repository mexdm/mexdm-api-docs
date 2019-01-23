MEXDM交易所官方API文档
==================================================
[MEXDM][]交易所开发者文档([English Docs][])。

<!-- TOC -->

- [介绍](#介绍)
- [开始使用](#开始使用)
- [API接口加密验证](#api接口加密验证)
    - [生成API Key](#生成api-key)
    - [发起请求](#发起请求)
    - [签名](#签名)
    - [选择时间戳](#选择时间戳)
    - [请求交互](#请求交互)
        - [请求](#请求)
        - [分页](#分页)
    - [标准规范](#标准规范)
        - [时间戳](#时间戳)
        - [例子](#例子)
        - [数字](#数字)
        - [限流](#限流)
                - [REST API](#rest-api)
- [永续(perpetual)业务API参考](#永续perpetual业务api参考)
    - [永续行情API](#永续行情api)
        - [1. 获取所有币对列表](#1-获取所有币对列表)
        - [2. 获取币对交易深度](#2-获取币对交易深度)
        - [3. 获取币对Ticker](#3-获取币对ticker)
        - [4. 获取币对历史成交记录](#4-获取币对历史成交记录)
        - [5. 获取K线数据](#5-获取k线数据)
        - [6. 获取服务器时间](#6-获取服务器时间)
    - [币币账户API](#币币账户api)
        - [1. 获取账户信息](#1-获取账户信息)
        - [2. 交易委托](#2-交易委托)
        - [3. 撤销所有委托](#3-撤销所有委托)
        - [4. 按订单撤销委托](#4-按订单撤销委托)
        - [5. 查询所有订单](#5-查询所有订单)
        - [6. 按id查询订单](#6-按id查询订单)
        - [7. 获取账单](#7-获取账单)

<!-- /TOC -->

# 介绍

欢迎使用[MEXDM][]开发者文档。

本文档提供了永续(Perpetual)业务的账户管理、行情查询、交易功能等相关API的使用方法介绍。
行情API提供市场的公开的行情数据接口，账户和交易API需要身份验证，提供下单、撤单，查询订单和帐户信息等功能。

# 开始使用    
REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如下：

+ 在RESTful架构中，每一个URL代表一种资源；
+ 客户端和服务器之间，传递这种资源的某种表现层；
+ 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。

建议开发者使用REST API进行币币交易或者资产提现等操作。

# API接口加密验证
## 生成API Key

在对任何请求进行签名之前，您必须通过 MEXDM 网站【用户中心】-【API】创建一个API key。 创建key后，您将获得3个必须记住的信息：
* API Key

* Secret Key

* Passphrase

API Key 和 Secret Key将由随机生成和提供，Passphrase由用户自己设定。

## 发起请求

所有REST请求都必须包含以下标题：

* ACCESS-KEY API KEY作为一个字符串。
* ACCESS-SIGN 使用base64编码签名（请参阅签名消息）。
* ACCESS-TIMESTAMP 作为您的请求的时间戳。
* ACCESS-PASSPHRASE 您在创建API密钥时设置的口令。
* 所有请求都应该含有application/json类型内容，并且是有效的JSON。

## 签名
ACCESS-SIGN的请求头是对 **timestamp + method + requestPath + "?" + queryString + body** 字符串(+表示字符串连接)使用 **HMAC SHA256** 方法加密，通过**BASE64** 编码输出而得到的。其中，timestamp 的值与 ACCESS-TIMESTAMP 
请求头相同。

* method 是请求方法(POST/GET/PUT/DELETE)，字母全部大写。
* requestPath 是请求接口路径。
* queryString GET请求中的查询字符串
* body 是指请求主体的字符串，如果请求没有主体(通常为GET请求)则body可省略。

**例如：对于如下的请求参数进行签名**

```bash
curl "https://www.mexdm.com/api/v1/perpetual/ccex/orders?limit=100"       
```
* 获取获取深度信息，以 fbtcusd 合约为例
```java
Method = "GET"
requestPath = "/api/v1/perpetual/public/products/fbtcusd/orderbook"
queryString= "?size=100"

```

生成待签名的字符串

```
GET /api/v1/perpetual/public/products/fbtcusd/orderbook?size=100'  
```
* 下单，以 fbtcusd 币对为例

```java
Method = "POST"
requestPath = "/api/v1/perpetual/products/fbtcusd/order"
body = {"type":10,"side":"open_long","price":100,"beMaker":0,"amount":1000}


```

生成待签名的字符串

```
POST /api/v1/perpetual/products/fbtcusd/order {"type":10,"side":"open_long","price":100,"beMaker":0,"amount":1000}'  
```

然后，将待签名字符串添加私钥参数生成最终待签名字符串。


例如：
```
Signature = hmac(secretkey, Message, SHA256)
```
在使用前需要对于Signature进行base64编码

```
Signature = base64.encode(Signature.digest())
```

## 请求交互  

REST访问的根URL：`https://www.mexdm.com`

### 请求

所有请求基于Https协议，请求头信息中Content-Type 需要统一设置为:'application/json’。

**请求交互说明**

1、请求参数：根据接口请求参数规定进行参数封装。

2、提交请求参数：将封装好的请求参数通过POST/GET/DELETE等方式提交至服务器。

3、服务器响应：服务器首先对用户请求数据进行参数安全校验，通过校验后根据业务逻辑将响应数据以JSON格式返回给用户。

4、数据处理：对服务器响应数据进行处理。

**成功**

HTTP状态码200表示成功响应，并可能包含内容。如果响应含有内容，则将显示在相应的返回内容里面。

**常见错误码**

* 400 Bad Request – Invalid request forma 请求格式无效

* 401 Unauthorized – Invalid API Key 无效的API Key

* 403 Forbidden – You do not have access to the requested resource 请求无权限

* 404 Not Found 没有找到请求

* 429 Too Many Requests 请求太频繁被系统限流

* 500 Internal Server Error – We had a problem with our server 服务器内部错误

* 如果失败，response body 带有错误描述信息

## 标准规范

### 时间戳

除非另外指定，API中的所有时间戳均以微秒为单位返回。

请求签名中的ACCESS-TIMESTAMP的单位是秒，允许用小数表示更精确的时间。请求的时间戳必须在API服务时间的30秒内，否则请求将被视为过期并被拒绝。如果本地服务器时间和API服务器时间之间存在较大的偏差，那么我们建议您使用通过查询API服务器时间来更新http header。

### 例子

1524801032573

### 数字

为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。 

整数（如交易编号和顺序）不加引号。

### 限流

如果请求过于频繁系统将自动限制请求，并在http header中返回429 too many requests状态码。

##### REST API

* 公共接口：我们通过IP限制公共接口的调用：每2秒最多10个请求。

* 私人接口：我们通过用户ID限制私人接口的调用：每2秒最多10个请求。

* 某些接口的特殊限制在具体的接口上注明

# 永续(perpetual)业务API参考

## 永续行情API

### 1. 获取所有合约列表

**HTTP请求**

```http
    # Request
    GET /v1/perpetual/public
```
```javascript
    # Response
    [
            {
                "amount24":"0.0000000000000000",
                "code":"fbtcusd",
                "env":0,
                "fluctuation":"0.0000",
                "fund":"0.0000000000000000",
                "high":"100.0000000000000000",
                "indexPrice":"100",
                "low":"100.0000000000000000",
                "markPrice":"100.0000000000000000",
                "price":"100",
                "size24":"0.0000000000000000",
                "totalPosition":"71355.0000000000000000"
            },
            ...
    ]
```

**返回值说明**  


|返回字段 | 字段说明|
| ----------|:-------:|
| amount24            | 24小时成交量|
| code   | 合约code |
| fluctuation  | 涨跌幅 |
| fund   | 资金费率 |
| high   | 最高价 |
| indexPrice | 指数价格 |
| low | 最低价  |
| markPrice | 标记价格 |
| price | 最新价 |
| size24 | 24小时成交价值 |
| totalPosition | 总持仓量 |

### 2. 获取币对交易深度

    获取币对盘口深度的请求列表。

**HTTP请求**

```http
    # Request
    GET /api/v1/perpetual/public/products/<code>/orderbook
```
```javascript
    # Response
    {
            "asks":[
                [
                    "88.96475",
                    "4460",
                    "14460"
                ],
                ...
            ],
            "bids":[
                [
                    "87.21174999",
                    "4500",
                    "14500"
                ],
                 ...
            ]
        }
```
**返回值说明**  


|返回字段|字段说明|  
| ------------- |----|
| asks | 卖方深度 |
| bids | 买方深度 |

**深度字段说明**  

|返回字段|字段说明|  
| ------------- |----|
| 87.21174999 | 价格 |
| 4500 | 总数量 |
| 14500 | 该价格前的总数量 |
**请求参数**  


| 参数名 | 参数类型  | 必填 | 描述 |
| ------------- |----|----|----|
| Code | String | 是 | 合约, 如 fbtcusd |

### 3. 获取合约Ticker

**HTTP请求**

    最新成交、24h最高、24h最低和24h成交量的快照信息。

```http
    # Request
    GET /v1/perpetual/public/products/tickers
```

```javascript
    # Response
    [
        1545722115925,
        "3797",
        "3700",
        "29620",
        "7.8172",
        "1",
        "3793",
        "3793",
        "0.00",
        "3792",
        "3795",
        "fbtcusd"
    ]
```

**返回值说明（从上到下按顺序)**

 
|返回字段|字段说明|
|--------| :-------: |
|时间戳| 1545722115925 |
|24h 最高|3797|
|24h 最低|3700|
|24h成交张数|29620|
|24h成交价值|7.8172|
|原始成交价|1
|最新成交价|3793|
|24小时价格涨跌幅|3793|
|24小时价格涨跌幅比例|0.00|
|盘口买一|3792|
|盘口卖一|3795|
|合约|fbtcusd|
    
    
**请求参数**

|参数名|参数类型|必填|描述|
|------|----|:---:|:---:|
|code|String|是|合约,如 fbtcusd|
    
### 4. 获取合约历史成交记录

    获取所请求合约的历史成交信息。

**HTTP请求**
```http
    # Request
    GET /api/v1/perpetual/public/fbtcusd/fills
```
```javascript
    # Response
    [
        [
            "100",
            "10",
            "short",
            1543309609577,
            1
        ],
        ...
    ]
```

**返回值说明（按顺序）**


|返回字段|字段说明|
|--------|----|
|成交价格 |100|
|成交量 |10|
|Maker成交方向|short|
|成交时间戳| 1543309609577|
|交易编号| 1|

**请求参数**

 |参数名|参数类型|必填|描述|
|-----|:---:|----|----|
|code|String|是|币对，如fbtcusd|

**解释说明**

  + 交易方向 side 表示每一笔成交订单中 maker 下单方向,maker 是指将订单挂在订单深度列表上的交易用户，即被动成交方。

  + short 代表做空，因为 maker 是买单，maker 的买单被成交，所以价格下跌；相反的情况下，short代表行情上涨，因为此时maker是卖单，卖单被成交，表示上涨。

### 5. 获取K线数据

**HTTP请求**

```http
    # Request
    GET  /api/v1/perpetual/public/fbtcusd/candles?type=1min&since=since
```
    
```javascript
    # Response
    {
        [1543405500000,"3980","4000","3984","3989","18675"]
        ...
    }
```

**返回值说明（按顺序）**  
    
|返回字段|字段说明|
|-----|----|
|K线开始时间戳|1543405500000|
|最低价|3980|
|最高价|4000|
|开盘价（第一笔交易）|3984|
|收盘价（最后一笔交易）|3989|
|收盘价（最后一笔交易）|3989|
|成交量|18675|

**请求参数**
    
|参数名|参数类型|必填|描述|
|-----|----|----|----|
|code|String|是|币对如fbtcusd|
|type|String|是|K线周期类型如1min/1hour/day/week/month|
|since|String|否|时间戳,默认值0|

### 6. 获取服务器时间

    获取API服务器的时间的接口。此接口不需要身份验证。

**HTTP请求**
```http
    # Request
    
    GET /api/v1/perpetual/public/time
```
    
```javascript
    # Reponse

    {
        "iso": "2015-01-07T23:47:25.201Z",
        "epoch": 1524801032573
    }
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|iso|为iso 8061标准的时间字符串表达的服务器时间|
|epoch|时间戳形式表达的服务器时间|


    iso：返回值为iso 8061标准的时间字符串  
    epoch：返回值为时间戳

## 合约账户API

### 1. 获取账户信息

    获取永续交易账户余额列表，查询各币种的余额，冻结和可用情况

**HTTP请求**

```
    # Request
    GET /api/v1/perpetual/account/assets
```
```
    # Response
    [
        {
            "availableBalance":"0",
            "contractCode":"",
            "currencyCode":"BTC",
            "env":0,
            "exchange":false,
            "orderMargin":"0",
            "positionMargin":"0",
            "realizedSurplus":"0",
            "rechargeable":true,
            "transfer":1,
            "withdrawable":true
        },
        ...
    ]
```

**返回值说明**

|返回字段|字段说明|
|----|----|
|availableBalance|可用资金|
|contractCode|合约代码|
|currencyCode|币种代码|
|env| 是否测试币 0:线上币,1:测试币|
|exchange|充值：false：不是充值货币，true：充值货币|
|orderMargin| 委托保证金|
|positionMargin| 仓位保证金|
|realizedSurplus| 已实现盈亏|
|rechargeable| 充值：false：不是充值货币，true：充值货币|
|transfer| 可划转 0:不可划转，1 可划转|
|withdrawable| 提现：false：不是提现货币，true:提现货币|

### 2. 交易委托

    MEXDM 提供限价/市价和条件委托两种订单类型。

**HTTP请求**
```
    # Request

    POST /api/v1/perpetual/products/fbtcusd/order
```

```javascript
    # Response

    {
        "id": 123456
    }
```
    
**返回值说明**

    + id: 订单ID

**请求参数**

|参数名| 参数类型 |必填|描述|
|:----:|:----:|:---:|----|
|type|String|是|10:限价委托,11:市价委托|
|code|String|是|币对如btc_usdt|
|side|String|是|open_long:开多 open_short:开空 close_long:平多 close_short:平空|
|amount|Double|否|发出限价委托以及市价卖出委托时传递，代表交易币的数量|
|price|Double|否|发出限价委托时传递，代表币对价格|
|beMaker|Integer|否| 被动委托：0:不care 1:只做maker，如果是taker就撤单
|triggerBy|String|否| 计划委托类型：index:指数价格 mark:标记价格 last:最新价格
|triggerPrice|Double|否| 触发价格

**解释说明**

  + triggerBy 只有是委托成交时传三个枚举值其中之一,这个值会作为是否条件委托单的依据.

### 3. 撤销所有委托

    撤销合约下所有未成交委托,由于是异步撤单所以该接口没有返回值。

**HTTP请求**
```
    # Request
    DELETE /api/v1/perpetual/products/fbtcusd/orders
```
```javascript
    # Response

    { ...}
```

**请求参数**

|参数名|参数类型|必填|描述|
|----|----| ----| ----|
|code|String|是|合约, 如 fbtcusd|

### 4. 按订单撤销委托

    按照订单id撤销指定订单,由于是异步撤单所以该接口没有返回值。

**HTTP请求**

```http
    # Request
    DELETE /api/v1/perpetual/products/fbtcusd/{orderId}
```
```javascript
    # Response
    {...}
```

**请求参数**

|参数名|参数类型|必填|描述|
|---|----|----|----|
|code|String|是|币对,如 fbtcusd|
|orderId|String|是|需要撤销的未成交委托的id（注：url 中的 orderId，如 ：/api/v1/perpetual/products/p_btc_usdt/order/1000）

### 5. 查询所有订单，支持分页查询

    按照订单状态查询所有订单。
    
**HTTP请求**

```http   
    # Request
    GET api/v1/perpetual/products/fbtcusd/list
```
```javascript
    # Response
    {
        "amount":1300,
        "avgPrice":0,
        "createdDate":1543827039000,
        "dealAmount":0,
        "orderSize":10,
        "price":130,
        "side":"short",
        "status":0
    }
```

**返回值说明**

|返回字段|字段说明|
|----|----|
|amount|委托数量|
|avgPrice|平均成交价格|
|createDate|创建订单的时间戳|
|dealAmount|订单已成交数量|
|orderSize|委托价值|
|price|订单委托价|
|side|下单方向，long多，short空|
|status|订单状态 0 等待成交 1 部分成交 2 已经成交 3 撤销中 -1 已经撤销|

**请求参数**

|参数名 | 参数类型 | 必填 | 描述 |
|---|----|----|----|
| code|String|是|币对如fbtcusd|

### 6. 按id查询订单

    按照订单id查询指定订单。

**HTTP请求**
```http
    # Request
    GET api/v1/perpetual/products/fbtcusd/{orderId}
```
```javascript
    # Response 
    {
        "amount":1300,
        "avgPrice":0,
        "createdDate":1543827039000,
        "dealAmount":0,
        "orderSize":10,
        "price":130,
        "side":"short",
        "status":0
    }
```

**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|amount|订单委托数量|
|avgPrice|订单已成交部分均价，如果未成交则为0|
|createDate|创建订单的时间戳|
|filledVolume|订单已成交数量|
|funds|订单已成交金额|
|orderSize|委托价值|
|price|订单委托价|
|side|下单方向，long多，short空|
|status|订单状态 0 等待成交 1 部分成交 2 已经成交 3 撤销中 -1 已经撤销|

**请求参数**  
    
|参数名|参数类型|必填|描述|
|-----|----|----|----|
|code|String|是|币对，如 fbtcusd|
|orderId|String|是|订单Id|

### 7. 获取账单，支持分页查询

    获取币币交易账户账单

**HTTP请求**
```http
    # Request
    GET /api/v1/perpetual/account/{currencyCode}/ledger
```
```javascript
    # Response
    {
        "code":0,
        "data":{
            "bills":[
                {
                    "amount":"0",
                    "balance":"1.00014",
                    "createdDate":1547524801000,
                    "currencyCode":"fbtc",
                    "detailSide":"long",
                    "fee":"0",
                    "feeCurrencyCode":"",
                    "profit":"0.00541916",
                    "size":"0",
                    "type":19
                },
                {
                    "amount":"0",
                    "balance":"1.00344276",
                    "createdDate":1547424566000,
                    "currencyCode":"fbtc",
                    "detailSide":"",
                    "fee":"0",
                    "feeCurrencyCode":"",
                    "profit":"1",
                    "size":"0",
                    "type":21
                }
            ],
            "paginate":{
                "page":1,
                "pageSize":50,
                "total":34
            }
        },
        "msg":"success"
    }

```

**返回值说明**

|返回字段 | 字段说明 |
|----|----|
|amount|账单发生数量|
|balance|账单资产余额|
|createdDate|账单发生时间戳|
|currencyCode|币种|
|detailSide|交易类型 1.开多open_long 2.开空open_short 3.平多close_long 4.平空close_short|
|fee|手续费: 正表示付手续费,负表示得手续费|
|feeCurrencyCode|手续费对应的币种，可能是币种，可能是点卡|
|profit|该笔交易对应的盈亏: 正表示盈利,负表示亏损|
|size|成交金额|
|type|11.充值 12.提现 13.转入 14.转出 15.多/买 16.空/卖 17.系统收取手续费 18.保险金 19.结算 20.穿仓对敲|

**请求参数**  
    
|参数名|参数类型|必填|描述|
|----|---|---|---|
|currencyCode|String|是| 币种代码，如fbtc|
|startDate|Long|否|开始时间|
|endDate|Long|否|结束时间|
|page|Integer|否|页码|
|limit|Integer|否|请求返回数据量，默认最大值 100|


[MEXDM]: https://www.mexdm.com 
[English Docs]: https://github.com/mexdm/mexdm-api-docs/blob/master/README_EN.md
[Unix Epoch]: https://en.wikipedia.org/wiki/Unix_time
