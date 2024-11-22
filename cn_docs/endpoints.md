
Starlette 包含了 `HTTPEndpoint` 和 `WebSocketEndpoint` 类，它们为处理 HTTP 方法分发和 WebSocket 会话提供了基于类的视图模式。

### HTTPEndpoint

`HTTPEndpoint` 类可以作为一个 ASGI 应用来使用：

```python
from starlette.responses import PlainTextResponse
from starlette.endpoints import HTTPEndpoint


class App(HTTPEndpoint):
    async def get(self, request):
        return PlainTextResponse(f"Hello, world!")
```

如果你使用 Starlette 应用实例来处理路由，可以将请求分发到 `HTTPEndpoint` 类。确保分发到类本身，而不是类的实例：

```python
from starlette.applications import Starlette
from starlette.responses import PlainTextResponse
from starlette.endpoints import HTTPEndpoint
from starlette.routing import Route


class Homepage(HTTPEndpoint):
    async def get(self, request):
        return PlainTextResponse(f"Hello, world!")


class User(HTTPEndpoint):
    async def get(self, request):
        username = request.path_params['username']
        return PlainTextResponse(f"Hello, {username}")

routes = [
    Route("/", Homepage),
    Route("/{username}", User)
]

app = Starlette(routes=routes)
```

HTTP 端点类对于任何没有对应处理器的请求方法，将会返回 "405 Method not allowed" 响应。

### WebSocketEndpoint

`WebSocketEndpoint` 类是一个 ASGI 应用，提供了一个包装器，封装了 `WebSocket` 实例的功能。

可以通过 `.scope` 在端点实例中访问 ASGI 连接范围，并且它有一个 `encoding` 属性，您可以选择设置此属性，以便在 `on_receive` 方法中验证预期的 WebSocket 数据。

支持的编码类型有：

* `'json'`
* `'bytes'`
* `'text'`

有三个可重写的方法来处理特定的 ASGI WebSocket 消息类型：

* `async def on_connect(websocket, **kwargs)`
* `async def on_receive(websocket, data)`
* `async def on_disconnect(websocket, close_code)`

```python
from starlette.endpoints import WebSocketEndpoint


class App(WebSocketEndpoint):
    encoding = 'bytes'

    async def on_connect(self, websocket):
        await websocket.accept()

    async def on_receive(self, websocket, data):
        await websocket.send_bytes(b"Message: " + data)

    async def on_disconnect(self, websocket, close_code):
        pass
```

`WebSocketEndpoint` 也可以与 `Starlette` 应用类一起使用：

```python
import uvicorn
from starlette.applications import Starlette
from starlette.endpoints import WebSocketEndpoint, HTTPEndpoint
from starlette.responses import HTMLResponse
from starlette.routing import Route, WebSocketRoute


html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""

class Homepage(HTTPEndpoint):
    async def get(self, request):
        return HTMLResponse(html)

class Echo(WebSocketEndpoint):
    encoding = "text"

    async def on_receive(self, websocket, data):
        await websocket.send_text(f"Message text was: {data}")

routes = [
    Route("/", Homepage),
    WebSocketRoute("/ws", Echo)
]

app = Starlette(routes=routes)
```
