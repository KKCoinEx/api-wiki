# api-wiki
KKCOIN.COM 以 RESTful 和 WebSocket 两种方式提供 API 给专业用户更高效的服务。目前，公开的交易数据通过 WebSocket 提供，不需要进行身份验证，私有数据则需要通过 HTTPS RESTful 方式访问，并且需要在每次访问时提交消息的签名以供服务器验证身份。要使用 KKCOIN.COM的 RESTful API，你需要提供令牌(API KEY)，路由（ENDPOINT），参数（PAYLOAD），专用字段（HEADER）和时间戳（TIMESTAMP）

### API KEY
KKCOIN.COM 通过令牌（API KEY）识别用户的身份和访问权限, 登入“用户中心”，“API” 栏目的“管理”，上传您的 RSA 公钥，即可生成并管理新的令牌，我们仅根据令牌去验证经过您的私钥签名的来访，KKCOIN.COM 不保留用户的私钥，请妥善保管本地的私钥文件，了解关于生成密钥的[更多信息]

### ENDPOINT
路由（[ENDPOINT](endpoint.md)）是用户通过 API 访问 KKCOIN.COM 具体功能的节点名称，目前开放的 ENDPOINT 有 [BALANCE](balance.md)（账户余额）, [ORDER](order.md)（委托状态）, [OPENORDERS](openorder.md)（当前委托）, [TRADE](trade.md)（交易）, [CANCEL](cancel.md)（取消委托）

### PAYLOAD
KKCOIN.COM 遵循 HTTP RESTful 原则，每个 HTTP API 路由有不同的参数和访问方式，通过 GET 方式访问获取信息路由参数拼装在 URL 字符串中，更新信息的路由则需要通过 POST 方式在 HTTP 请求的 BODY 部分传递

### TIMESTAMP
UNIX 系统通常用始于 1970 年 1 月 1 日标准时间（UTC）的一个秒数计数器来跟踪时间，KKCOIN.COM API 服务器会检查来自用户的请求是否和当前时间一致，并要求用户对时间戳进行签名，以保证访问的真实性。考虑到网络和服务器的差异，一个合理的误差是被允许的。

### SIGNATURE
路由，参数和时间戳三个因子，构成了一次完整的用户访问描述，为了保证用户访问的真实、不可篡改和不可盗用，KKCOIN.COM 要求用户对这三个因子同时用 RSA 算法生成的私钥通过 SHA256 算法进行签名并转码成 Base64 编码格式，一个签名只能被使用一次，因此同样的指令如果在一秒钟之内被重复提交也会被拒绝。出于对用户资产安全的重视，KKCOIN.COM 的加密机制较其他交易所复杂。我们为开发了 [PHP DEMO](https://github.com/KKCoinEx/api-wiki/wiki/PHP-DEMO) 和 [Node.JS DEMO](nodejs_demo.md) ，用户可以通过查阅 DEMO 节省开发时间和了解 KKCOIN.COM 的安全机制
