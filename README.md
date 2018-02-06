# KKCOIN.COM api-wiki
KKCOIN.COM 以 RESTful 和 WebSocket 两种方式提供 API 服务给专业用户。

公开的交易数据通过 [WebSocket 提供](https://github.com/KKCoinEx/api-wiki/wiki/WebSocket-API)，不需要进行身份验证。

私有数据则需要通过 [HTTPS RESTful](https://github.com/KKCoinEx/api-wiki/wiki/RESTful--API), 方式访问，并且需要在每次访问时提交消息签名以供验证。
