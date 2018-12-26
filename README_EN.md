# KKCoin交易所官方API文档

KKCoin.COM provides automated data and transation services to profesisonal users in both RESTful and WebSocket formats.

<!-- TOC -->

- [Introduction](#Introduction)
- [Getting Started](#Getting Started)
- [Encrypted Verification of API](#api接口加密验证)
  - [Generate Key](#生成秘钥)
    - [Upload public key](#上传公钥)
    - [Compilation package](#打包报文)
    - [Private Key Signature](#私钥签名)
    - [Compile http](#打包http)
- [Spot API Reference](#币币业务api参考)
  - [Spot Market API](#币币行情api)
    - [1. Access list of all Tickers](#1-获取交易对Ticker)
    - [2. Access market trading records of trading pairs](#2-获取交易对成交记录)
    - [3. Access depth table of trading pairs](#3-获取交易对交易深度)
    - [4. Access Candlestick chart](#4-获取K线数据)
  - [Spot account API](#币币账户api)
    - [1. Access accout information](#1-查询账户信息)
    - [2. Order status](#2-查询订单状态)
    - [3. Search fulfilled orders](#3-查询有效委托)
    - [4. Order placement](#4-委托下单)
    - [5. Cancel orders](#5-取消委托)
  - [WebSocketAPI](#WebSocketAPI)

<!-- /TOC -->

#  Introduction

Welcome to KKcoin API document for developers.

This document provides instructions on how to use APIs related to account management, spot trading, market information, trading functions etc of spot trading activity.


# Getting started
REST, a.k.a. Representational State Trasnfer, is an architectural style that defines a set of constraints and properties based on HTTP. REST is known for its clear structure, readability, standardization and scalability. Its advantages are as follows:

- Each URL represents one web resource in RESTful architecture;
- Acting as a representation of resources between client and server;
- Client is enabled to operate server-side resources with 4 HTTP requests - representational state transfer.

Developers are recommended to use REST API to proceed spot trading and withdrawals. 

# Encrypted Verification of API
![Client key generation and signature process](https://github.com/KKCoinEx/api-wiki/blob/master/chart/rest_auth_client.png)

## Generate key

**NOTE**：To use KKCOIN.COM API ，users need to verify that the OpenSSL / LibreSSL tool is available locally.

Enter the host command line tool, select a folder with the file to be written and execute the following command:

```bash
$ openssl
OpenSSL> genrsa -aes256 -out private.pem 2048
OpenSSL> rsa -in private.pem -pubout -out public.pem 
OpenSSL> exit
```

After the second command line, the system will ask for a password to encrypt the private key. Please write down the password which will be used to subsequent oeprations and signatures.

| File generated       | Details                                    |
| ----------- | --------------------------------------- |
| private.pem | Developer RSA private key, please keep it safe                        |
| public.pem  | Open with a text edit and submit content to KKCOIN.COM to extract API Key |

While file type is .pem in this example, it is not madatory

## Upload public key

Open KKCOIN.COM  **Account** then **API**, then **Settings**

![API](https://github.com/KKCoinEx/api-wiki/blob/master/chart/apientrance.png)

In the API Settings page

![api Setting](https://github.com/KKCoinEx/api-wiki/blob/master/chart/apikey.png)

Enter a reference name in the small box on the right. Generate the lublic key from the previous step, copy its contents and paste it into the input box and click on **Create New KEY** to get an API KEY per below example

```
490a3730e367de19ab16b9ede63a71f8
```

Note：You are uploading the **Public Key**，NOT the private key. Please DO NOT share the private key with anyone that has no authority to operate your account. KKCOIN.COM will NEVER ask you for your private key.

## Compilation package
### Route

The name of API interface for query, for eg. use 'balance' to query for balance

### Parameter set
This refers to an ordered array of parameters of the routing node (Endpoint). In order to ensure that the server and the client compiles consistently, we require the parameter set to be sorted according to the first ASCII code of values (sorted in alphabetical order). If the same character occurs, the second ASCII code of values is implemented and so on. The sorted parameter set(array) is encoded into a JSON format string. The parameter segment (payload) is detailed in [Address and Routing parameters](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-url-&-endpoint)

### Timestamp
The timestamp is a way to track time in a UNIX system with seconds counter starting at 1 Jan, 1970 Standard Time (UTC). KKCOIN.COM servers will check if timestamp requested is consistent with servers. Once verified, the user is required to sign the timestamp to ensure authenticity of access, otherwise it will be rejected. There may be a difference between times betweent the network and server; the server will accept a reasonable timeframe variation.

### Package

Once the Route, Parameter set and timestamp are connected, the message can be signed,

Example:
<pre>
trade{"amount":"10","orderop":"BUY","ordertype":"LIMIT","price":"0.01","symbol":"EOS_ETH"}1517921954
</pre>
**NOTE**

- Parameter values of numeric types are converted to string types in advance
- The assembly order must be Route + Parameter Set + Timestamp, otherwise signature verification will fail
- Partial routes such as balance and openorders do not need parameters，as the server will encode an empty array（array）as square brackets []

Example
```
balance[]1518016060
```

## Private key signature
After message packet is signed, the signature is completed by the signature module in the OpenSSL library in local development environment. KKCOIN.COM requires the SHA256 algorithm to sign and convert to Base64 format. The PHP implementation code is as follows:

```php
# Create an OpenSSL recourse by protecting the password and private key
$openssl_res = openssl_get_privatekey($this->private_key_file, $this->passphrase);

# To sign packet $message using OPENSSL_ALGO_SHA256 algorithm，write $signature_bin
$ssl_res = openssl_sign($message, $signature_bin, $openssl_res, OPENSSL_ALGO_SHA256);

# Convert $signature_bin in binary format to base64 format
$signature = base64_encode($signature_bin);
```

The resulting signature string might look like this

<pre>
agsXB/yyeijqV4HFJepo3OPQc1H07otzWTEoeGNR+BF+jI7tkmhizgZWWFUhjELtSd1+9QrghkpuIV0Kkyqy0ph8CDpOM9jkpuXhbkK5z3rYMdIfVMQPkVIEk9EPpaY+D3k39SnFS7fPiJ***********delete for security ************************WpiJRuek9o7jNUpOw1n+zeK3b9a3Bp2Ps8qCeUlN8Hk9XTWZXWpuUvorf0VEnqhsj9F+hcss7PMoc9bRGE++adtd46Id/ARtltHaW0KYBdQn2G9wJjB1EMzEdMvQNzF/W35i4XSMbOmQ=
</pre>
Note：A signature can only be used once, so the same instruction will be rejected if submitted a second time.

## Compile http
THe user needs three fields in the HTTP header sent to KKCOIN.COM

| Name           | Input                      |
| --------------- | ------------------------ |
| KKCOINAPIKEY    | Get API KEY from KKCoin.COM |
| KKCOINSIGN      | Signature string of packet               |
| KKCOINTIMESTAMP | Timestamp when request is sent               |

Example

```
GET /rest/trade? HTTP/1.1
Host: api.kkcoin.com:80
KKCOINAPIKEY: 1f614ce13a2bebab9c6d24ead4f5ec8f
KKCOINSIGN: lhd+0b4yKmmuN7zSuUhyKzHQobgBLBbKDGjtdN2W0d2zu2w/IxC8qjN5+bQ7xVE8QScJSju/jCLs+n8d5C6ufzJTWfUQkj0gxXYBmIu4/HtxcNYQ6oVjc+x7PjPWgfQTbpy4siPwG7JkN7CvW+pTE00A4JRFQRCZyxoyZMki4d4zjdAs+XPXKVsrb0oICkMit5I5DjZc9mMIS1B3JAmQHwvtWeOljZit4v+MHuM6Js7oLGXd/piuMLO+8JB9IwInnMCZniFddbuEtxOWkoJUddPBlT/XPWEsqMBnsbmTH5EE9Gs/dkDk76mjTSpkoC39b40o6HR0EHlEsjdz5k0iiA== 
KKCOINTIMESTAMP: 1517629282
Cache-Control: no-cache
```

**Note**  Without these three fields, the server will not be able to process request.

When complete, user can complete the API access via CURL or other similar libraries

**SUCCESS**
HTTP code status 200 indicates success and may include content which will be displayed in the result. 

**Common code errors**
| status_code | message                                  | reason                                   |
| ----------- | ---------------------------------------- | ---------------------------------------- |
| Client side        |                                          |                                          |
| 400         | General client mistake.                  | Incorrect parameter passed |
| 481         | HTTP HEADER lacks the necessary keys.    | Missing HTTP HEADER|
| 482         | Timestamp verification failed.           | HTTP HEADER includes invalid timestamp|
| 483         | Signature duplicates are not allowed.    | HTTP HEADER includes used signature|
| 484         | Access authorization verification failed. | Current API KEY does not have user's authorization. A common situation arises when a read-only API key accesses the transaction route. |
| 485         | IP rejected.                             | Current IP violates user set IP LIMIT policy               |
| 486         | message' => 'Invalid API-KEY.            | Wrong or deleted API KEY                        |
| 487         | Signature verification failed.           | If signature verification fails, it is more complicated. May be caused by incorrect packet format, incorrect key pair, timestamp and/or HEADER timestamp |
| Server side         |                                          |                                          |
| 500         | General server failure.                  | Server cannot identify error, please contact KKCOIN.COM                |
| 581         | OpenSSL report an error occurred.        | Serve signature verification error, please contact KKCOIN.COM                 |
| 582         | Database report an error occurred.       | Server database error, please contact KKCOIN.COM                  |

# Spot API reference
## Spot market API
### 1-Access list of all Tickers
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
### 2-Access market trading records of trading pairs
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
### 3-Access depth table of trading pairs
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
### 4-Access Candlestick chart
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
## Spot account API
### 1-Access accout information
Query type GET
**GET:** https://api.kkcoin.com/rest/balance

| Parameter   | Details   |
| ---- | ---- |
| None    |      |

**Return**

| Field            | Details     |
| ------------- | ------ |
| asset_symbol  | Asset symbol   |
| bal           | Balance    |
| available_bal | Available balance |
| frozen_bal    | Locked balance |

### 2-Query order status
Query type GET
**GET:** https://api.kkcoin.com/rest/order

| Parameter   | Details    |
| ---- | ----- |
| id   | Order ID |

**Return**

| Field     | Details                                        |
| ------ | ---------------------------------------- |
| order_id        | Order ID                                      |
| symbol          | Trading pair，eg：KK_ETH                          |
| type            | Order type，LIMIT / 限价单，MARKET / 市价单（No support currently）      |
| orderop         | BUY / 买，SELL / 卖                         |
| price           | Order price                                    |
| origin_amount   | Order lots                                     |
| executed_price  | Average price executed                                     |
| executed_amount | Transacted Lots                                      |
| executed_quote_amount | Transacted amount                                      |
| status          | Order status： NEW / 新委托单，FILLED / 已完成，PARTIALLY_FILLED / 部分成交，CANCELED / 已取消 |

### 3-Search fulfilled orders
Query type GET
**GET:** https://api.kkcoin.com/rest/openorders

| Parameter     | Details              |
| ------ | --------------- |
| symbol | Trading pair，eg：KK_ETH |

**Return**

| Field              | Detail                                      |
| --------------- | ---------------------------------------- |
| order_id        | Order ID                                      |
| symbol          | Trading pair，eg：KK_ETH                          |
| type            | Order type，LIMIT / 限价单，MARKET / 市价单（暂不支持）      |
| orderop         | BUY / 买，SELL / 卖                         |
| price           | Order price                                     |
| origin_amount   | Order Lots                                     |
| executed_price  | Average price executed                                     |
| executed_amount | Transacted Lots                                      |
| status          | Order status：NEW / 新委托单， FILLED / 已完成， PARTIALLY_FILLED / 部分成交，CANCELED / 已取消 |
| source          | Order source                                     |
| ip              | Order IP address                              |

### 4-Order placement
Query type POST
**POST:** https://api.kkcoin.com/rest/trade
**Parameter**

| Field        | Details               |
| --------- | ---------------- |
| symbol    | Trading pair，eg：KK_ETH  |
| ordertype | Order type，LIMIT / 限价单 |
| orderop   | BUY / 买，SELL / 卖 |
| price     | Order price             |
| amount    | Order Lots             |

**Return**

| Field       | Details   |
| -------- | ---- |
| order_id | Order ID  |

**Note**

Returning Order ID does not mean that order is successful. Order status must be obtaining by routing order status via the appropriate route to get confirmation
### 5-Cancel orders
Query type POST
**POST:** https://api.kkcoin.com/rest/cancel

| Field   | Details   |
| ---- | ---- |
| id   | Order ID  |

**Return**

| Field       | Details     |
| -------- | ------ |
| order_id | Order ID cancelled |

**Note:**
Returning Order ID does not mean that order is successful. Order status must be obtaining by routing order status via the appropriate route to get confirmation

## WebSocketAPI

[WebSocketAPI](https://github.com/KKCoinEx/api-wiki/wiki/WebSocket-API)
