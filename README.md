# kkcoin.com api-wiki
KKCOIN.COM 以 RESTful 和 WebSocket 两种方式提供 API 给专业用户更高效的服务。目前公开的交易数据通过 WebSocket 提供，不需要进行身份验证，私有数据则需要通过 HTTPS RESTful 方式访问，并且需要在每次访问时提交消息的签名以供验证。要使用 KKCOIN.COM的 RESTful API，你需要提供令牌(API KEY)，路由（ENDPOINT），参数（PAYLOAD），专用字段（HEADER）和时间戳（TIMESTAMP）

### API KEY
KKCOIN.COM 通过令牌（API KEY）识别用户的身份和访问权限, 登入“用户中心”，“API” 栏目的“管理”，上传您的 RSA 公钥，即可生成并管理新的令牌。我们根据令牌和经过您的私钥签名验证来访的真实性，KKCOIN.COM 不保留用户的私钥，请妥善保管本地的私钥文件，了解关于生成密钥的[更多信息](https://github.com/KKCoinEx/api-wiki/wiki/RSA-%E5%AF%86%E9%92%A5%E7%94%9F%E6%88%90)

### ENDPOINT
路由（[ENDPOINT](https://github.com/KKCoinEx/api-wiki/wiki/API-RESTful-endpoint)）是用户通过 API 访问 KKCOIN.COM 具体功能的节点名称，目前开放的 RESTful endpoint 有 [BALANCE](https://github.com/KKCoinEx/api-wiki/wiki/API-RESTful-endpoint#balance)（账户余额）, [ORDER](https://github.com/KKCoinEx/api-wiki/wiki/API-RESTful-endpoint#order)（委托状态）, [OPENORDERS](https://github.com/KKCoinEx/api-wiki/wiki/API-RESTful-endpoint#openorders)（当前委托）, [TRADE](https://github.com/KKCoinEx/api-wiki/wiki/API-RESTful-endpoint#trade)（交易）, [CANCEL](https://github.com/KKCoinEx/api-wiki/wiki/API-RESTful-endpoint#cancel)（取消委托）

### PAYLOAD
KKCOIN.COM 遵循 HTTP RESTful 原则，每个 HTTP API 路由有不同的参数和访问方式，通过 GET 方式访问获取信息路由参数拼装在 URL 字符串中，更新信息的路由则需要通过 POST 方式在 HTTP 请求的 BODY 部分传递，KKCOIN.COM 以 Json 序列化的方式对 Payload 数组进行打包，实现方法可以参考具体的语言 Deom

### TIMESTAMP
时间戳是 UNIX 系统中用始于 1970 年 1 月 1 日标准时间（UTC）的一个秒数计数器来跟踪时间的方式，KKCOIN.COM 服务器会检查来自用户请求中的时间戳是否和服务器时间一致，并要求用户对时间戳进行签名，以保证访问的真实性，不在当前时间发起的访问会被拒绝。考虑到网络和服务器的差异，一个合理的误差是被允许的。

### SIGNATURE
路由，参数和时间戳三个因子，构成了一次完整的用户访问描述，为了保证用户访问指令的不可篡改和不可盗用，KKCOIN.COM 要求用户同时对这三个因子用 RSA 算法私钥以 SHA256 算法进行签名并转码成 Base64 编码格式，一个签名只能被使用一次，因此同样的指令如果在一秒钟之内被重复提交也会被拒绝。

出于对用户资产安全的重视，KKCOIN.COM 的加密机制较其他交易所复杂。我们为开发了 [PHP DEMO](https://github.com/KKCoinEx/api-wiki/wiki/PHP-DEMO) 和 [Node.JS DEMO](https://github.com/KKCoinEx/api-wiki/wiki/demo-Node.js) ，用户可以通过查阅 DEMO 节省开发时间和了解 KKCOIN.COM 的安全机制
