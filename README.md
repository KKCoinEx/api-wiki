# KKCOIN.COM api-wiki
KKCOIN.COM 以 RESTful 和 WebSocket 两种方式提供 API 服务给专业用户。公开的交易数据通过 WebSocket 提供，不需要进行身份验证，私有数据则需要通过 HTTPS RESTful 方式访问，并且需要在每次访问时提交消息签名以供验证。

使用 KKCOIN.COM 的 RESTful API 时需要提供令牌(API KEY)，路由（endpoint），参数（payload），和时间戳（timestamp）

### api key
KKCOIN.COM 通过令牌（API KEY）识别用户的身份和访问权限, 登入“用户中心”，“API” 栏目的“管理”，上传您的 RSA 公钥，即可生成并管理新的令牌。我们根据令牌和经过您的私钥签名验证来访的真实性，KKCOIN.COM 不保留用户的私钥，请妥善保管本地的私钥文件，了解关于生成密钥的[更多信息](https://github.com/KKCoinEx/api-wiki/wiki/Authentication)

### endpoint
路由 (endpoint) 是用户通过 API 访问 KKCOIN.COM 具体功能的节点名称，目前开放的 RESTful endpoint 有 [balance](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-::-balance)（查询账户余额）, [order](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-::-order)（查询委托状态）, [openorders](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-::-openorder)（查询当前委托）, [trade](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-::-trade)（委托交易）, [cancel](https://github.com/KKCoinEx/api-wiki/wiki/RESTful-::-cancel)（取消委托）

### payload
KKCOIN.COM 遵循 HTTP RESTful 原则，每个 HTTP API 路由有不同的参数和访问方式，通过 GET 方式访问获取信息的路由，参数拼装在 URL 字符串中，通过 POST 方式更新信息的路由，则需要通过 HTTP 请求的 BODY 部分传递参数，KKCOIN.COM 以 Json 序列化的方式对 Payload 数组进行打包，可以参考 [PHP Demo](https://github.com/KKCoinEx/api-php-demo) 和 [Node.JS Demo](https://github.com/KKCoinEx/api-nodejs-demo)的实现方法

### timestamp
时间戳是 UNIX 系统中用始于 1970 年 1 月 1 日标准时间（UTC）的一个秒数计数器来跟踪时间的方式，KKCOIN.COM 服务器会检查来自用户请求中的时间戳是否和服务器时间一致，并要求用户对时间戳进行签名以保证访问的真实性，携带错误的时间戳发起的访问会被拒绝，考虑到网络和服务器的差异，服务器可以接受合理的时间误差。

### signature
路由(endpoint)，参数(payload)和时间戳(timestamp)三个因子，构成了一次完整的用户访问行为描述，为了保证用户访问指令的不可篡改和不可盗用，KKCOIN.COM 要求用户同时对这三个因子用私钥以 SHA256 算法进行签名并转码成 Base64 编码格式，一个签名只能被使用一次，因此同样的指令如果在一秒钟之内被重复提交也会被拒绝。

KKCOIN.COM 出于对用户资产安全的重视，制定了相较于大多数数字货币交易所更复杂的加密机制。为了协助用户快速熟悉和利用 API，我们开发了 [PHP Demo](https://github.com/KKCoinEx/api-php-demo) 和 [Node.JS Demo](https://github.com/KKCoinEx/api-nodejs-demo) ，您可以通过查阅 Demo 文档及源代码了解 KKCOIN.COM 的安全机制及 API 使用方法
