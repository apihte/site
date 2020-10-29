# Node.js 路由

当 http 服务器接收到一个请求，这个请求中会包含请求的 URL、GET/POST 参数，路由需要根据这些参数来执行相应的代码。

我们需要的所有数据都包含在 request 对象中，该对象作为 onRequest() 回调函数的第一个参数传递。为了解析这些数据，我们需要额外的 Node.js 模块，url 和 querystring 模块。

```
   url.parse(string).pathname     url.parse(string).query
                        |            |
                      -----	------------------- 
http://localhost:8888/start?foo=bar&hello=world
                                ---       -----
                                 |	        |
querystring.parse(queryString)["foo"]     querystring.parse(queryString)["hello"]
```

