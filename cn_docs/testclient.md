
测试客户端允许您使用 `httpx` 库向 ASGI 应用程序发出请求。

```python
from starlette.responses import HTMLResponse
from starlette.testclient import TestClient


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = HTMLResponse('<html><body>Hello, world!</body></html>')
    await response(scope, receive, send)


def test_app():
    client = TestClient(app)
    response = client.get('/')
    assert response.status_code == 200
```

测试客户端暴露了与其他 `httpx` 会话相同的接口。
特别需要注意的是，请求调用只是标准的函数调用，而不是可等待对象。

您可以使用 `httpx` 标准 API 中的任何功能，例如身份验证、会话 cookie 处理或文件上传。

例如，要在 `TestClient` 上设置头部，您可以这样做：

```python
client = TestClient(app)

# 为未来的请求设置头部
client.headers = {"Authorization": "..."}
response = client.get("/")

# 为每个请求单独设置头部
response = client.get("/", headers={"Authorization": "..."})
```

例如，向 `TestClient` 发送文件：

```python
client = TestClient(app)

# 发送单个文件
with open("example.txt", "rb") as f:
    response = client.post("/form", files={"file": f})

# 发送多个文件
with open("example.txt", "rb") as f1:
    with open("example.png", "rb") as f2:
        files = {"file1": f1, "file2": ("filename", f2, "image/png")}
        response = client.post("/form", files=files)
```

有关更多信息，请查看 `httpx` [文档](https://www.python-httpx.org/advanced/)。

默认情况下，`TestClient` 会引发应用程序中发生的任何异常。偶尔，您可能希望测试 500 错误响应的内容，而不是允许客户端引发服务器异常。在这种情况下，您应该使用 `client = TestClient(app, raise_server_exceptions=False)`。

!!! note

    如果您希望 `TestClient` 运行 `lifespan` 处理程序，
    您需要将 `TestClient` 作为上下文管理器使用。它不会在 `TestClient` 实例化时触发。您可以在 [这里](lifespan.md#running-lifespan-in-tests) 了解更多。

### 选择异步后端

`TestClient` 接受 `backend`（字符串类型）和 `backend_options`（字典类型）参数。这些选项会传递给 `anyio.start_blocking_portal()`。有关接受的后端选项的更多信息，请参阅 [anyio 文档](https://hellowac.github.io/anyio-zh-cn/basics.html#backend-options)。默认情况下，`asyncio` 被使用，并且使用默认选项。

要使用 `Trio` 后端，传递 `backend="trio"`。例如：

```python
def test_app():
    with TestClient(app, backend="trio") as client:
       ...
```

要使用 `asyncio` 和 `uvloop`，传递 `backend_options={"use_uvloop": True}`。例如：

```python
def test_app():
    with TestClient(app, backend_options={"use_uvloop": True}) as client:
       ...
```

### 测试 WebSocket 会话

您还可以使用测试客户端测试 WebSocket 会话。

`httpx` 库将用于构建初始握手，这意味着您可以在 http 和 WebSocket 测试之间使用相同的身份验证选项和其他头部。

```python
from starlette.testclient import TestClient
from starlette.websockets import WebSocket


async def app(scope, receive, send):
    assert scope['type'] == 'websocket'
    websocket = WebSocket(scope, receive=receive, send=send)
    await websocket.accept()
    await websocket.send_text('Hello, world!')
    await websocket.close()


def test_app():
    client = TestClient(app)
    with client.websocket_connect('/') as websocket:
        data = websocket.receive_text()
        assert data == 'Hello, world!'
```

会话上的操作是标准的函数调用，而不是可等待对象。

使用 `with` 上下文管理器块内的会话非常重要。这确保了 ASGI 应用程序所在的后台线程被正确终止，并且应用程序中发生的任何异常都始终由测试客户端引发。

#### 建立测试会话

* `.websocket_connect(url, subprotocols=None, **options)` - 接受与 `httpx.get()` 相同的参数集。

如果应用程序没有接受 WebSocket 连接，则可能引发 `starlette.websockets.WebSocketDisconnect` 异常。

`websocket_connect()` 必须作为上下文管理器使用（在 `with` 块中）。

!!! 注意
    `websocket_connect` 不支持 `params` 参数。如果您需要传递查询参数，请直接将其硬编码到 URL 中。

    ```python
    with client.websocket_connect('/path?foo=bar') as websocket:
        ...
    ```

#### 发送数据

* `.send_text(data)` - 向应用程序发送给定的文本。
* `.send_bytes(data)` - 向应用程序发送给定的字节数据。
* `.send_json(data, mode="text")` - 向应用程序发送给定的数据。使用 `mode="binary"` 以二进制数据帧发送 JSON 数据。

#### 接收数据

* `.receive_text()` - 等待应用程序发送的文本，并返回它。
* `.receive_bytes()` - 等待应用程序发送的字节串，并返回它。
* `.receive_json(mode="text")` - 等待应用程序发送的 JSON 数据，并返回它。使用 `mode="binary"` 以二进制数据帧接收 JSON 数据。

可能会引发 `starlette.websockets.WebSocketDisconnect` 异常。

#### 关闭连接

* `.close(code=1000)` - 执行客户端关闭 WebSocket 连接。

### 异步测试

有时，您可能需要在应用程序外执行异步操作。例如，您可能希望在使用现有的异步数据库客户端/基础设施调用应用程序后检查数据库的状态。

对于这些情况，使用 `TestClient` 可能会很困难，因为它创建了自己的事件循环，而异步资源（如数据库连接）通常无法跨事件循环共享。
解决这个问题的最简单方法是使整个测试异步，并使用异步客户端，如 [httpx.AsyncClient]。

以下是一个此类测试的示例：

```python
from httpx import AsyncClient
from starlette.applications import Starlette
from starlette.routing import Route
from starlette.requests import Request
from starlette.responses import PlainTextResponse


def hello(request: Request) -> PlainTextResponse:
    return PlainTextResponse("Hello World!")


app = Starlette(routes=[Route("/", hello)])


# 如果使用 pytest，您需要添加一个异步标记，例如：
# @pytest.mark.anyio  # 使用 https://github.com/agronholm/anyio
# 或安装并配置 pytest-asyncio (https://github.com/pytest-dev/pytest-asyncio)
async def test_app() -> None:
    # 注意：您必须为相对 URL（如 "/"）设置 `base_url` 才能正常工作
    async with AsyncClient(app=app, base_url="http://testserver") as client:
        r = await client.get("/")
        assert r.status_code == 200
        assert r.text == "Hello World!"
```

[httpx.AsyncClient]: https://www.python-httpx.org/advanced/#calling-into-python-web-apps
