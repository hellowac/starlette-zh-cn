
Starlette 提供了一个 `WebSocket` 类，其功能类似于 HTTP 请求，但允许通过 WebSocket 进行数据的发送和接收。

---

### WebSocket

签名：`WebSocket(scope, receive=None, send=None)`

```python
from starlette.websockets import WebSocket


async def app(scope, receive, send):
    websocket = WebSocket(scope=scope, receive=receive, send=send)
    await websocket.accept()
    await websocket.send_text('Hello, world!')
    await websocket.close()
```

WebSocket 提供了一个映射接口，因此可以像使用 `scope` 一样使用它。

例如：`websocket['path']` 将返回 ASGI 路径。

---

#### URL

WebSocket 的 URL 可以通过 `websocket.url` 访问。

该属性实际上是 `str` 的子类，并且还提供了可以从 URL 中解析出的所有组件。

例如：`websocket.url.path`、`websocket.url.port`、`websocket.url.scheme`。

---

#### Headers

请求头以不可变的大小写不敏感多重字典形式暴露。

例如：`websocket.headers['sec-websocket-version']`

---

#### 查询参数（Query Parameters）

查询参数以不可变的多重字典形式暴露。

例如：`websocket.query_params['search']`

---

#### 路径参数（Path Parameters）

路由器路径参数以字典接口的形式暴露。

例如：`websocket.path_params['username']`

---

### 接受连接

* `await websocket.accept(subprotocol=None, headers=None)`  

该方法用于接受 WebSocket 连接，可选参数包括：

- **`subprotocol`**：指定子协议的名称。
- **`headers`**：附加自定义头信息作为字典传入。

### 发送数据

* `await websocket.send_text(data)`
* `await websocket.send_bytes(data)`
* `await websocket.send_json(data)`

从版本 0.10.0 开始，JSON 消息默认通过文本数据帧发送。
要通过二进制数据帧发送 JSON，可以使用 `websocket.send_json(data, mode="binary")`。

### 接收数据

* `await websocket.receive_text()`
* `await websocket.receive_bytes()`
* `await websocket.receive_json()`

可能会引发 `starlette.websockets.WebSocketDisconnect()` 异常。

从版本 0.10.0 开始，JSON 消息默认通过文本数据帧接收。
要通过二进制数据帧接收 JSON，可以使用 `websocket.receive_json(data, mode="binary")`。

### 数据迭代

* `websocket.iter_text()`
* `websocket.iter_bytes()`
* `websocket.iter_json()`

这些方法与 `receive_text`、`receive_bytes` 和 `receive_json` 类似，但返回的是异步迭代器。

```python hl_lines="7-8"
from starlette.websockets import WebSocket


async def app(scope, receive, send):
    websocket = WebSocket(scope=scope, receive=receive, send=send)
    await websocket.accept()
    async for message in websocket.iter_text():
        await websocket.send_text(f"Message text was: {message}")
    await websocket.close()
```

当引发 `starlette.websockets.WebSocketDisconnect` 异常时，迭代器将退出。

### 关闭连接

* `await websocket.close(code=1000, reason=None)`

### 发送和接收消息

如果你需要发送或接收原始的 ASGI 消息，应该使用 `websocket.send()` 和 `websocket.receive()`，而不是使用原始的 `send` 和 `receive` 可调用对象。这样可以确保 WebSocket 的状态保持正确更新。

* `await websocket.send(message)`
* `await websocket.receive()`

### 发送拒绝响应

如果在调用 `websocket.accept()` 之前调用 `websocket.close()`，则服务器会自动向客户端发送 HTTP 403 错误。

如果你想发送其他错误响应，可以使用 `websocket.send_denial_response()` 方法。该方法会发送响应并关闭连接。

* `await websocket.send_denial_response(response)`

这要求 ASGI 服务器支持 WebSocket 拒绝响应扩展。如果不支持，将引发 `RuntimeError` 异常。
