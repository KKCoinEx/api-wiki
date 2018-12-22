# KKCoin交易所官方API文档

KKCoin.COM 以 RESTful 和 WebSocket 两种方式为专业用户提供自动化的数据及交易服务

<!-- TOC -->

- [介绍](#介绍)
- [开始使用](#开始使用)
- [API接口加密验证](#api接口加密验证)
  - [生成秘钥](#生成秘钥)
    - [上传公钥](#上传公钥)
    - [打包报文](#打包报文)
    - [私钥签名](#私钥签名)
    - [打包http](#打包http)
- [币币业务API参考](#币币业务api参考)
  - [币币行情API](#币币行情api)
    - [1. 获取交易对Ticker](#1-获取交易对Ticker)
    - [2. 获取交易对成交记录](#2-获取交易对成交记录)
    - [3. 获取交易对交易深度](#3-获取交易对交易深度)
    - [4. 获取K线数据](#4-获取K线数据)
  - [币币账户API](#币币账户api)
    - [1. 查询账户信息](#1-查询账户信息)
    - [2. 查询订单状态](#2-查询订单状态)
    - [3. 查询有效委托](#3-查询有效委托)
    - [4. 委托下单](#4-委托下单)
    - [5. 取消委托](#5-取消委托)

<!-- /TOC -->

#  介绍

欢迎使用KKCoin.COM开发者文档。

本文档提供了币币业务的账户管理、行情查询、交易功能等相关API的使用方法介绍。 行情API提供市场的公开的行情数据接口，账户和交易API需要身份验证，提供下单、撤单，查询订单和帐户信息等功能。

# 开始使用
REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如下：

- 在RESTful架构中，每一个URL代表一种资源
- 客户端和服务器之间，传递这种资源的某种表现层
- 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”

建议开发者使用REST API进行币币交易等操作。
# API接口加密验证
![客户端密钥生成及签名流程](https://github.com/KKCoinEx/api-wiki/blob/master/chart/rest_auth_client.png)

## 生成秘钥

**注意**：要使用 KKCOIN.COM API ，用户需要确认在本地具有可用的 OpenSSL / LibreSSL 工具。

进入主机命令行工具，选择一个具有写入文件的文件夹，执行下列命令

```bash
$ openssl
OpenSSL> genrsa -aes256 -out private.pem 2048
OpenSSL> rsa -in private.pem -pubout -out public.pem 
OpenSSL> exit
```

第二行命令之后，系统会要求输入一个密码，用于对私钥进行加密，请记下密码，在后续操作及签名过程中会用到

| 生成的文件       | 说明                                      |
| ----------- | --------------------------------------- |
| private.pem | 开发者RSA私钥，请务必妥善保管                        |
| public.pem  | 用文本编辑器打开，将内容提交给 KKCOIN.COM 用于提取 API KEY |

公钥文件内容示例，文件名后缀通常为 PEM，但这不是强制要求

## 上传公钥

打开 KKCOIN.COM 进入 **用户中心** 找到 **API 模块**，点击 **管理**

![API](https://github.com/KKCoinEx/api-wiki/blob/master/chart/apientrance.png)

进入 API 管理页面

![api 管理](https://github.com/KKCoinEx/api-wiki/blob/master/chart/apikey.png)

在右边小框中输入一个助记名称，打开上一步生成公钥文件，复制全部内容，粘贴到下面输入框中，点击**创建新KEY**，即可获得一个如下例子的 API KEY

```
490a3730e367de19ab16b9ede63a71f8
```

注意：上传的是**公钥**，不是私钥，不要将私钥交给任何未得到您授权可以访问及操作您的账户的人和机构，KKCOIN.COM 也不会向您索取私钥

## 打包报文
### 路由

要访问的 API 接口的名称，比如查询余额的名称是 balance

### 参数集合
参数集合指的是路由节点（Endpoint）的参数组成的有序数组，为了保障服务端和客户端对参数集合的打包一致，我们要求参数集合按照键值的第一个字符ASCII码递增排序（字母升序排序），如果遇到相同字符则按照第二个字符的键值ASCII码递增排序，以此类推，将排序后的参数集合（数组）编码为 JSON 格式的字符串，即得到消息报文的参数段（payload）详见[地址和路由参数](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-url-&-endpoint)

### 时间戳
时间戳（timestamp）是 UNIX 系统中用始于 1970 年 1 月 1 日标准时间（UTC）的一个秒数计数器来跟踪时间的方式，KKCOIN.COM 服务器会检查来自用户请求中的时间戳是否和服务器时间一致，并要求用户对时间戳进行签名以保证访问的真实性，携带错误的时间戳发起的访问会被拒绝，考虑到网络和服务器的差异，服务器可以接受合理的时间误差

### 拼装消息报文（Package）

路由名称、参数段、时间戳三个字符串连接起来，就是可以进行签名的消息报文

例如
<pre>
trade{"amount":"10","orderop":"BUY","ordertype":"LIMIT","price":"0.01","symbol":"EOS_ETH"}1517921954
</pre>
**注意**

- 数字类型的参数值要提前转化为字符串型
- 组装顺序必需是 路由+参数段+时间戳，否则会导致签名验证失败
- 部分路由如 balance 和 openorders 不需要携带参数，在服务端会将一个空的数组（array）编码为空的方括号 []

例如
```
balance[]1518016060
```

## 私钥签名
完成消息报文打包后，利用本地开发环境中 OpenSSL 库中的签名模块完成签名，KKCoin.COM 要求用以 SHA256 算法进行签名并转成 Base64 编码格式，以下为 PHP 的实现代码：

```php
# 通过保护私钥文件的密码和私钥文件创建 OpenSSL 资源
$openssl_res = openssl_get_privatekey($this->private_key_file, $this->passphrase);

# 用 OPENSSL_ALGO_SHA256 算法对消息报文 $message 签名，写入 $signature_bin
$ssl_res = openssl_sign($message, $signature_bin, $openssl_res, OPENSSL_ALGO_SHA256);

# 将二进制格式的 $signature_bin 转换成 base64 编码
$signature = base64_encode($signature_bin);
```

得到的签名字符串可能是下面这个样子的

<pre>
agsXB/yyeijqV4HFJepo3OPQc1H07otzWTEoeGNR+BF+jI7tkmhizgZWWFUhjELtSd1+9QrghkpuIV0Kkyqy0ph8CDpOM9jkpuXhbkK5z3rYMdIfVMQPkVIEk9EPpaY+D3k39SnFS7fPiJ***********delete for security ************************WpiJRuek9o7jNUpOw1n+zeK3b9a3Bp2Ps8qCeUlN8Hk9XTWZXWpuUvorf0VEnqhsj9F+hcss7PMoc9bRGE++adtd46Id/ARtltHaW0KYBdQn2G9wJjB1EMzEdMvQNzF/W35i4XSMbOmQ=
</pre>
注意：一个签名只能被使用一次，因此同样的指令如果在一秒钟之内被重复提交也会被拒绝

## 打包http
用户在发送到 KKCOIN.COM 的 HTTP HEADER 中，需要携带三个字段

| 字段名称            | 填入内容                       |
| --------------- | ------------------------ |
| KKCOINAPIKEY    | 从 KKCoin.COM 获取的 API KEY |
| KKCOINSIGN      | 消息报文的签名字符串               |
| KKCOINTIMESTAMP | 发送请求时的时间戳                |

例如

```
GET /rest/trade? HTTP/1.1
Host: api.kkcoin.com:80
KKCOINAPIKEY: 1f614ce13a2bebab9c6d24ead4f5ec8f
KKCOINSIGN: lhd+0b4yKmmuN7zSuUhyKzHQobgBLBbKDGjtdN2W0d2zu2w/IxC8qjN5+bQ7xVE8QScJSju/jCLs+n8d5C6ufzJTWfUQkj0gxXYBmIu4/HtxcNYQ6oVjc+x7PjPWgfQTbpy4siPwG7JkN7CvW+pTE00A4JRFQRCZyxoyZMki4d4zjdAs+XPXKVsrb0oICkMit5I5DjZc9mMIS1B3JAmQHwvtWeOljZit4v+MHuM6Js7oLGXd/piuMLO+8JB9IwInnMCZniFddbuEtxOWkoJUddPBlT/XPWEsqMBnsbmTH5EE9Gs/dkDk76mjTSpkoC39b40o6HR0EHlEsjdz5k0iiA== 
KKCOINTIMESTAMP: 1517629282
Cache-Control: no-cache
```

**注意**  如果没有这三个字段，服务器将无法解析用户的请求

完成后，就可以通过 curl 或者其他类似的库完成 API 访问了

# 币币业务API参考
## 币币行情API
### 1-获取交易对Ticker
The ticker shows you the current best bid and ask, last trade price, price change, daily volume and so on.

**GET:** https://api.kkcoin.com/rest/allticker

```
[
  [
    "KK_ETH",       // Symbol
    "0.00000523",   // Best bid price
    "30582",        // Best bid amount
    "0.00000526",   // Best ask price
    "321179",       // Best ask amount
    "-0.00000005",  // Price change of last 24 hours
    "-0.0095",      // Price change rate
    "0.00000522",   // Last trade price 
    "232.88",       // Daily volume
    "0.00000530",   // Daily high
    "0.00000495",   // Daily low
    "46576000.00"   // Daily amount
  ],
  ...
]
```
### 2-获取交易对成交记录
Get recent trades (up to last 100), include price, amount and time.

**GET:** https://api.kkcoin.com/rest/trades?symbol=<symbol\>

```
[
  [
    1519365636000,  // Timestamp
    "0.00000526",   // Price
    "6419",         // Amount
    true,           // Maker is buyer
    "20181010000000626386" // Trade ID
  ],
  ...
]
```
### 3-获取交易对交易深度
Get current order book (up to 200).

**GET:** https://api.kkcoin.com/rest/book?symbol=<symbol\>

```
{
  "a":               // Ask
  [
    [
      "0.00000528",  // Price
      "14778"        // Amount
    ],
    ...
  ],
  "b":               // Bid
  [
    [
      "0.00000526",  // Price
      "26602"        // Amount
    ],
    ...
  ]
}
```
### 4-获取K线数据
Get recent kline (up to last 200).

**Time interval**
* 1m: one minute
* 5m : five minutes
* 15m : 15 minutes
* 30m : 30 minutes
* 1h : one hour
* 2h : 2 hours
* 4h : 4 hours
* 6h : 6 hours
* 12h : 12 hours
* 1d : one day
* 1w : one week

**GET:** https://api.kkcoin.com/rest/kline?symbol=<symbol\>&interval=<interval\>

```
[
  [
    1518274800000,  // Timestamp
    "0.00000560",   // Open
    "0.00000550",   // Close
    "0.00000600",   // High
    "0.00000550",   // Low
    "12400"         // Volume
  ],
  ...
]
```
## 币币账户API
### 1-查询账户信息
访问方式 GET

| 参数   | 说明   |
| ---- | ---- |
| 无    |      |

#### 返回

| 字段            | 说明     |
| ------------- | ------ |
| asset_symbol  | 资产符号   |
| bal           | 余额     |
| available_bal | 其中可用金额 |
| frozen_bal    | 其中冻结金额 |

### 2-查询订单状态
访问方式 GET

| 参数   | 说明    |
| ---- | ----- |
| id   | 订单 ID |

#### 返回

| 字段     | 说明                                       |
| ------ | ---------------------------------------- |
| order_id        | 订单号                                      |
| symbol          | 交易对符号，例如：KK_ETH                          |
| type            | 委托类型，LIMIT / 限价单，MARKET / 市价单（暂不支持）      |
| orderop         | BUY / 买，SELL / 卖                         |
| price           | 委托价格                                     |
| origin_amount   | 委托数量                                     |
| executed_price  | 成交均价                                     |
| executed_amount | 成交量                                      |
| executed_quote_amount | 成交额                                      |
| status          | 订单状态： NEW / 新委托单，FILLED / 已完成，PARTIALLY_FILLED / 部分成交，CANCELED / 已取消 |

### 3-查询有效委托
访问方式 GET

| 参数     | 说明              |
| ------ | --------------- |
| symbol | 交易对符号，例如：KK_ETH |

#### 返回

| 字段              | 说明                                       |
| --------------- | ---------------------------------------- |
| order_id        | 订单号                                      |
| symbol          | 交易对符号，例如：KK_ETH                          |
| type            | 委托类型，LIMIT / 限价单，MARKET / 市价单（暂不支持）      |
| orderop         | BUY / 买，SELL / 卖                         |
| price           | 委托价格                                     |
| origin_amount   | 委托数量                                     |
| executed_price  | 成交均价                                     |
| executed_amount | 成交量                                      |
| status          | 订单状态：NEW / 新委托单， FILLED / 已完成， PARTIALLY_FILLED / 部分成交，CANCELED / 已取消 |
| source          | 订单来源                                     |
| ip              | 订单来源 IP 地址                               |

### 4-委托下单
访问方式 POST

#### 参数

| 字段        | 说明               |
| --------- | ---------------- |
| symbol    | 交易对符号，例如：KK_ETH  |
| ordertype | 委托类型，LIMIT / 限价单 |
| orderop   | BUY / 买，SELL / 卖 |
| price     | 委托价格             |
| amount    | 委托数量             |

#### 返回

| 字段       | 说明   |
| -------- | ---- |
| order_id | 订单号  |

**注意**

得到返回订单号不代表下单成功，需要通过 order 路由查询订单状态确认执行的结果
### 5-取消委托
访问方式 POST

| 参数   | 说明   |
| ---- | ---- |
| id   | 订单号  |

#### 返回

| 字段       | 说明     |
| -------- | ------ |
| order_id | 被取消订单号 |

**注意**
得到返回订单号不代表取消成功，需要通过 order 路由查询订单状态确认执行的结果
