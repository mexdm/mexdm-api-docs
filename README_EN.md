MEXDM-official-api-docs
==================================================
[MEXDM][]Developer Documentation([English Docs][])

<!-- TOC -->

- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Encrypted Verification of API](#encrypted-verification-of-api)
    - [Generate an API Key](#generate-an-api-key)
    - [Initiate a Request](#initiate-a-request)
    - [Signature](#signature)
    - [Select timestamp](#select-timestamp)
    - [Request Process](#request-process)
        - [Request](#request)
        - [Pagination](#pagination)
    - [Standards and Specification](#standards-and-specification)
        - [Timestamp](#timestamp)
        - [For example,](#for-example)
        - [Numbers](#numbers)
        - [Rate Limits](#rate-limits)
            - [REST API](#rest-api)
- [Perpetual API Reference](#perpetual-api-reference)
    - [Perpetual Market API](#perpetual-market-api)
        - [1. Access the list of all trading pairs](#1-access-the-list-of-all-trading-pairs)
        - [2. Access the depth table of trading pairs](#2-access-the-depth-table-of-trading-pairs)
        - [3. Access the ticker of a trading pair](#3-access-the-ticker-of-a-trading-pair)
        - [4. Access the trading records of a trading pair](#4-access-the-trading-records-of-a-trading-pair)
        - [5. Access Candlestick chart](#5-access-candlestick-chart)
        - [6. Access Server Time](#6-access-server-time)
    - [Perpetual Account API](#perpetual-account-api)
        - [1. Access account information](#1-access-account-information)
        - [2. Order Placement](#2-order-placement)
        - [3. Cancel all orders](#3-cancel-all-orders)
        - [4. Cancel a specified order](#4-cancel-a-specified-order)
        - [5. Search orders](#5-search-orders)
        - [6. Order inquiry by Order ID](#6-order-inquiry-by-order-id)
        - [7. Access the account statement](#7-access-the-account-statement)

<!-- /TOC -->

# Introduction
Welcome to [MEXDM][] API document for developers.

This document provides instructions on how to use APIs related to account management, market information, trading functions among others in perpetual contract trading.

Market API provides market data that are accessible to the public. Account APIs and trading APIs, which provide functions such as order placement, order cancellation, order inquiry and account information, need identity authentication.

# Getting Started

REST, a.k.a Respresntational State Transfer, is an architectural style that defines a set of constraints and properties based on HTTP. REST is known for its clear structure, readability, standardization and scalability. Its advantages are as follows:

+ Each URL represents one web resource in RESTful architecture;
+ Acting as a representation of resources between client and server;
+ Client is enabled to operate server-side resources with 4 HTTP requests - representational state transfer.

Developers are recommended to use REST API to proceed withdrawals.

# Encrypted Verification of API
## Generate an API Key

Before signing any request, you must generate an API key via MEXDM’s official website 【User Center】- 【API】. After generating the key, there are three things you must bear in 
mind:
* API Key

* Secret Key

* Passphrase

API Key and Secret are randomly generated and provided, Passphrase is set by user.

## Initiate a Request

All REST requests must include the following headings:

* ACCESS-KEY API Key as a string.
* ACCESS-SIGN uses base64-encoded signatures (see Signed Messages).
* ACCESS-TIMESTAMP is the timestamp of your request.header MUST be number of seconds since [Unix Epoch][] in UTC. Decimal values are allowed.
* ACCESS-PASSPHRASE is the password you specified when you generate the API key. 
* All requests should contain content like application/json and be valid JSON.

## Signature

The ACCESS-SIGN header is the output generated by using HMAC SHA256 to create the HMAC SHA256 using the BASE64 decoding secret key in the prehash string to generate **timestamp + method + requestPath + "?" + queryString + body** (where ‘+’ represents the string concatenation) and BASE64 encoded output. The timestamp value is the same as the ACCESS-TIMESTAMP 
header.

* method (POST/GET/PUT/DELETE)，These methods should be capitalized.
* requestPath is the request interface path.
* queryString GET is the string during the request.
* body is the request body string or omitted if there is no request body (usually the GET request).

**For example, if we sign the following parameters**

```bash
curl "https://www.mexdm.com/api/v1/perpetual/ccex/orders?limit=100"       
```
* Get the orderbook information，e.g. fbtcusd contract 
```java
Method = "GET"
requestPath = "/api/v1/perpetual/public/products/fbtcusd/orderbook"
queryString= "?size=100"

```

Generate the string to be signed

```
GET /api/v1/perpetual/public/products/fbtcusd/orderbook?size=100'  
```
* Place order，e.g. fbtcusd contract 
```java
Method = "POST"
requestPath = "/api/v1/perpetual/products/fbtcusd/order"
body = {"type":10,"side":"open_long","price":100,"beMaker":0,"amount":1000}


```

Generate the string to be signed

```
POST /api/v1/perpetual/products/fbtcusd/order {"type":10,"side":"open_long","price":100,"beMaker":0,"amount":1000}'  
```

Then, the character to be signed is added with the private key parameters to generate the final character string to be signed.


For example：
```
Signature = hmac(secretkey, Message, SHA256)
```
Need to encode base64 before Signature

```
Signature = base64.encode(Signature.digest())
```

## Request Process 
  
The root URL for REST access：`https://www.mexdm.com`

### Request
All requests are based on Https protocol, contentType in the request header must be uniformly set as: ‘application/json’.

**Request Process Descriptions**

1. Request parameter: parameter encapsulation based on the port request.

2. Submitting request parameter: submit the encapsulated parameter request to the server via POST/GET/PUT/DELETE or other methods.

3. Server response: the server will first perform a security validation, then send back the requested data to the client in JSON format.

4. Data processing: processing server response data.

**Success**

HTTP status code 200 indicates a successful response and may contain content. If the response contains content, it will appear in the corresponding returned content.

**Common Error Code**

* 400 Bad Request – Invalid request format

* 401 Unauthorized – Invalid API Key

* 403 Forbidden – You do not have access to the requested resource

* 404 Not Found

* 429 Too Many Requests

* 500 Internal Server Error – We had a problem with our server

* If failed，response body contains wrong descriptive information.

## Standards and Specification

### Timestamp

Unless otherwise specified, all timestamps in APIs are returned in microseconds.

The ACCESS-TIMESTAMP header must be the number of seconds since UTC's time [Unix Epoch][]. Decimal values are allowed. 
Your timestamp must be within 30 seconds of the API service time, otherwise your request will be considered expired and rejected. If you think there is a large time difference between your server and the API server, then we recommend that you use the time point to check the API server time.

### For example

1524801032573

### Numbers

In order to maintain the accuracy of cross-platform, decimal numbers are returned as strings. We suggest that you might be better to convert the number to string when issuing the request to avoid truncation and precision errors. Integers (such as transaction number and sequence) do not need quotation marks.

### Rate Limits

When a rate limit is exceeded, a status of 429 Too Many Requests will be returned.

##### REST API

* Public interface: We limit the invocation of public interface via IP: up to 6 requests every 2s.

* Private interface: We limit the invocation of private interface via user ID: up to 6 requests every 2s.

* Special restrictions on specified interfaces are specified.

# Perpetual API Reference

## Perpetual Market API

### 1. Access the list of all trading pairs

**HTTP Request**

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

**Response Details**  


| Field | Descirption |
| ----------|:-------:|
| amount24            | 24H Volume|
| code   | Contract Code |
| fluctuation  | Change |
| fund   | Funding Rate |
| high   | Highest Price |
| indexPrice | Index Price |
| low | Lowest Price  |
| markPrice | Mark Price |
| price | Latest Price |
| size24 | 24H Transaction Value |
| totalPosition | Total Position |

### 2. Access the depth table of trading pairs


**HTTP Request**

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
**Response Details**  


|Field|Description|  
|---- |------------|
| asks | depth of sellers |
| bids | depth of buyers |

**Response Details of Depth**  

|Field|Description|   
| ------------- |----|
| 87.21174999 | Price |
| 4500 | Volume |
| 14500 | Total |
**Request Paramters**  


| Name | Type  | Requited | Description |
| ------------- |----|----|----|
| Code | String | Y | Trading Pair, e.g. fbtcusd |

### 3. Access the ticker of a trading pair

**HTTP Request**

    The snapshot of the latest price, the highest bid price, the lowest ask price and 24-hour trading volume.

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

**Response Details (from the top down)**
 
|Field|Description|
|--------| :-------: |
|Timestamp| 1545722115925 |
|24h Highest|3797|
|24h Lowest|3700|
|24h Contracts|29620|
|24h Value|7.8172|
|Original Price|1
|Latest Price|3793|
|24h Change|3793|
|24h Change Ratio|0.00|
|Bid 1|3792|
|Ask 1|3795|
|Trading Pair|fbtcusd|
    
    
**Request Parameter**

|Name|Type|Required|Description| 
|------|-----|-----|-----|
|code|String|Y|Trading Pair, e.g. fbtcusd|
    
### 4. Access the trading records of a trading pair

    Access the history records of requested contract

**HTTP Request**
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

**Response Description (In order)**


|Field|Description|
|--------|----|
|Execution Price |100|
|Volume |10|
|Maker Side|short|
|Timestamp| 1543309609577|
|Transaction ID| 1|

**Request Paramters**

|Name|Type|Required|Description| 
|-----|-----|-----|-----| 
|code|String|Y|Trading pair, e.g. fbtcusd|

 **Explanation**

  + Side indicates that the direction of the order the maker places. Maker refers to a trader who places orders in the market, a marker is a passive transaction party

  + Short suggests price fall and long suggests price rise.

### 5. Access Candlestick chart

**HTTP Request**

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

**Response Details (in order)** 
    
|Field|Description|
|-----|----|
|Start timestamp|1543405500000|
|The lowest price|3980|
|The highest price|4000|
|Opening price（First transaction）|3984|
|Closing price（Last transaction）|3989|
|Trading Volume|18675|

**Request parameters**
    
|Name|Type|Required|Description|
|-----|-----|-----|-----|
|code|String|Y|Trading pair, e.g.fbtcusd|
|type|String|Y|K-line Type, e.g.1min/1hour/day/week/month|
|since|String|N|Timestamp,Default 0|

### 6. Access Server Time

    Access API server time. This interface does not require ID authentication.

**HTTP Request**
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
    
**Response Description**
    
|Field|Description|  
|------|-----|  
|epoch|server time expressed in second|
|iso|server time expressed in time string by ISO 8061|
|timestamp|server time expressed in millisecond|


       iso: Response is returned in time string by ISO 8061
       epoch: Response is retured in timestamp

## Perpetual Account API

### ### 1. Access account information

    Access the list of balance, inquiry of coin balances, freezing status and available fund in perpetual account.

**HTTP Request**

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

**Response Details**

|Field|Description|
|----|----|
|availableBalance|Available Balance|
|contractCode|Contract Code|
|currencyCode|Currency Code|
|env| Test Token Or Not 0:Listed Token,1:Test Token|
|exchange|Deposit：false：Not Deposit Currency，true：Deposit Currency|
|orderMargin| Order Margin|
|positionMargin| Position Margin|
|realizedSurplus| Realized PnL|
|rechargeable| Deposit：false：Not Deposit Currency，true：Deposit Currency|
|transfer| Transferable 0:Untransferable，1 Transferable|
|withdrawable| Withdraw：false：Not Withdrawal Currency，true:Withdrawal Currency|

### 2. Order Placement

    There are two categories of orders that can be placed on MEXDM -- limit order and market order.

**HTTP Request**
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
    
**Response Details**

    + orderId: Order ID

**Request Paramters**

|Name| Type | Required | Description |
|:----:|:----:|:---:|----|
|type|String|Y|10:Limit Order,11:Market Order|
|code|String|Y|Trading pair, e.g.fbtcusd|
|side|String|Y|open_long:Open Long open_short:Open Short close_long:Close Long close_short:Close Short|
|amount|Double|N|delivered when a limit order or selling market order if placed,representing the number of coins for trading|
|price|Double|N|delivered when a limit order is placed, representing the price of the pair|
|beMaker|Integer|N| Post-Only：0:Doesn't care 1:Only maker，Cancelled if taker
|triggerBy|String|N| Trigger Type：index:Index Price mark:Mark Price last:Last Price
|triggerPrice|Double|N| Trigger Price

**Explanation**

  + triggerBy One of trigger prices will be delivered when order filled, that price will decide whether it is a conditional order.

### 3. Cancel all orders

    Cancel all unfilled orders of the target trading pair, no interface response due to asynchronous cancellation.

**HTTP Request**
```
    # Request
    DELETE /api/v1/perpetual/products/fbtcusd/orders
```
```javascript
    # Response

    { ...}
```

**Request Paramters**

|Name|Paramters|Type|Description|
|----|-----| -----| -----|
|code|String|Y|Trading pairs, e.g. fbtcusd|

### 4. Cancel a specified order

    Cancel a specified order by order ID,no interface response due to asynchronous cancellation.

**HTTP Request**

```http
    # Request
    DELETE /api/v1/perpetual/products/fbtcusd/{orderId}
```
```javascript
    # Response
    {...}
```

**Request Paramters**

|Name|Type|Required|Description|
|---|----|----|----|
|code|String|Y|Trading Pair, e.g. fbtcusd|
|orderId|String|Y|The ID of an unfilled order specified need to be cancelled|
（Note：orderId in url，e.g. ：/api/v1/perpetual/products/p_btc_usdt/order/1000）

### 5. Search orders

    Check all the orders by order status.
    
**HTTP Request**

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

**Response Details**

|Field|Description|
|----|----|
|amount|Order Amount|
|avgPrice|Average Price for the Filled Orders|
|createDate|Timestamp upon the placement of the order|
|dealAmount|The Volume of the Filled Orders|
|orderSize|Order Value|
|price|Price set for the order|
|side|Order direction，long，short|
|status|Order Status 0 Wait to be filled 1 Partially filled 2 filled 3 Cancelling -1 Cancelled|

**Request Paramters**

|Name | Type | Required | Description |
|------|-----|-----|-----|
|code|String|Y|Trading pair, e.g.fbtcusd|

### 6. Order inquiry by Order ID

    Inquiry of a specified order by order ID.

**HTTP Request**
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

**Response Details**
    
|Field|Description|
|-----|----|
|amount|Order Amount|
|avgPrice|Average Price for the Filled Orders,0 for the unfilled orders|
|createDate|Timestamp upon the placement of the order|
|filledVolume|The Volume of the Filled Orders|
|funds|The Amount of the Filled|
|orderSize|Order Value|
|price|Price set for the order|
|side|Order direction，long，short|
|status|Order Status 0 Wait to be filled 1 Partially filled 2 filled 3 Cancelling -1 Cancelled|

**Request Paramters**  
    
|Name|Type|Required|Description
|-----|----|----|----|
|code|String|Y|Trading pair, e.g. fbtcusd|
|orderId|String|Y|Order Id|

### 7. Access the account statement, pagination supported.

    Access the statement of a perpetual account

**HTTP Request**
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

**Response Details**

|Field | Description |
|----|----|
|amount|Trading Volume on the Statement|
|balance|Statement balance|
|createdDate|Timestamp on the statement taking place|
|currencyCode|Currency Type|
|detailSide|Trading Type 1.Open Long open_long 2.Open Short open_short 3.Close Long close_long 4.Close Short close_short|
|fee|fee: + pays fee,- gets fee|
|feeCurrencyCode|Currency for fee，Probably token，Probably loyalty point|
|profit|Profit and loss: + means profit,- means loss|
|size|Transaction Amount|
|type|11.Deposit 12.Withdraw 13.Transfer In 14.Transfer Out 15.Long/Buy 16.Short/Sell 17.System Charges Fee 18.Margin 19.Settle 20.Auto-deleverage|

**Request Paramters**  
    
|Name|Type|Required|Description|
|----|---|---|---|
|currencycode|String|Y| Trading pair, e.g.fbtc|
|startDate|Long|N|Start Date|
|endDate|Long|N|End Date|
|page|Integer|N|Page|
|limit|Integer|N|Request Data Volume，Default Max 100|


[MEXDM]: https://www.mexdm.com 
[English Docs]: https://github.com/mexdm/mexdm-official-api-docs/blob/master/README_EN.md
[Unix Epoch]: https://en.wikipedia.org/wiki/Unix_time