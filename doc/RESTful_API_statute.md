## 一、RESTful API 规约

### （一）URL 设计

1. 【推荐】对于 API的设计需要遵循“动词 + 宾语”的结构，比如，`GET /orders`。动词通常就是五种 HTTP 方法：GET(Read)、POST(Create)、PUT(Update)、PATCH(部分Update)、DELETE(Delete)，对应 CURD 操作；宾语就是 API 的 URL。
    > 正例：
    > - GET /orders：获得所有订单
    > - POST /orders：新建一张订单
    > - GET /orders/{id}：获取某张的订单信息
    > - PUT /orders/{id}: 更新某张的订单信息（提供该订单的全部信息）
    > - PATCH /orders/{id}：更新某张的订单信息（提供该订单的部分信息）
    > - DELETE /orders/{id}：删除某张订单
    > 
    > 反例：
    > - GET /getAllOrders
    > - POST /createNewOrder
    > - DELETE /deleteAllInvalidOrders

2. 【推荐】为了统一起见，建议都使用复数 URL，比如 `GET /orders/2` 要好于 `GET /order/2`。
    > 说明：对于 URL 名词使用复数还是单数，其实没有强制规定，但是常见的操作是读取一个集合，比如 `GET /orders`（获取所有订单），这里明显应该是复数。

3. 【推荐】避免多级URL
   > 反例：
   > - GET /orders/30/details/2
   > - GET /orders/finished
   >
   > 正例：
   > - GET /orders/30?details=2
   > - GET /orders?finished=true
   >
   > 说明：这种 URL 不利于扩展，语义也不明确，往往要想一会儿，才能明白含义。更好的做法是除了第一级，其他级别都用查询字符串表达。

### （二）返回处理

1. 【强制】调用 API 服务后返回数据采用统一格式，返回的 HTTP 状态码为 2xx，代表调用成功
   
   正例：
   ```json
    Request URL: http://localhost:8080/orders/1
    Request Method: GET
    Status Code: 200 
    {
        "requestId": "d176e2a28470445c9b793c1e918f55f1",
        "code": "0",
        "message": "OK",
        "result": {
            "comment": "Hello5, world!",
            "id": 1
        }
    }
   ```
   > 说明：调用成功不代表当前请求业务通过，仅仅代表该请求有正确且合法的响应。更多说明请看后面的规约。

2. 【强制】调用接口出错后，将不会返回结果数据，错误结果分为业务错误与系统错误，当用户访问 API 出错时，API 会返回给用户一个合适的 2xx，4xx 或者 5xx 的 HTTP 状态码；以及一个 text/xml 格式的消息体。错误响应的消息体例子如下：
   
   业务错误
   ```json
    Request URL: http://localhost:8080/courses?name=%E7%8E%8B%E4%BA%94
    Request Method: GET
    Status Code: 200 

    {
        "requestId": "7ae7c602c87242deb78d5ad6839d167e",
        "code": "hello.1001",
        "message": "该学生还未注册任何课程。"
    }
   ```
   系统错误
   ```json
    Request URL: http://localhost:8080/courses
    Request Method: GET
    Status Code: 400 

    {
        "requestId": "58fa6e9ddc3d443baa3524fddd4fb226",
        "code": "sys.MissingParameter",
        "message": "必填参数没有填，请检查调用时是否填写了此参数，并重试请求。 : name"
    }
   ```
   > 说明：
   > 
   > 所有错误的消息体中都包括以下元素：
   > - requestId: 用于唯一标识该次请求的编号；当你无法解决问题时，可以提供这个 RequestId 寻求 API 支持工程师的帮助。
   > - code：返回给用户的错误码。
   > - message：详细错误信息。
   >
   > 针对系统错误，开发者可以提供一套「公共错误码」；而对于业务错误，则需要各业务服务实现。

3. 【强制】对于获取集合的接口，当查不到集合数据时，应返回空数组。
   正例：
   ```json
    Request URL:  http://192.168.206.25:8080/courses?name=张三
    Request Method: GET
    Status Code: 200 
    {
        "requestId": "46e4f4c6f1544996bafd8ae302320e25",
        "code": "0",
        "message": "OK",
        "result": {
            "courses": []
        }
    }
   ```
   反例：
   ```json
    Request URL:  http://192.168.206.25:8080/courses?name=张三
    Request Method: GET
    Status Code: 200 
    {
        "requestId": "46e4f4c6f1544996bafd8ae302320e25",
        "code": "0",
        "message": "OK",
        "result": {
            "courses": null
        }
    }
   ```

4. 【强制】参数名统一使用 lowerCamelCase 风格，必须遵循驼峰形式。
   > 正例：localValue
   >
   > 反例：LocalValue
