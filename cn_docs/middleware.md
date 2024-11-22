
Starlette 包含了多个中间件类，用于为整个应用程序添加行为。这些中间件都是作为标准的 ASGI 中间件类实现的，可以应用于 Starlette 或任何其他 ASGI 应用程序。

## 使用中间件

Starlette 应用类允许您以确保它始终被异常处理程序包装的方式包含 ASGI 中间件。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware
from starlette.middleware.trustedhost import TrustedHostMiddleware

routes = ...

# 确保所有请求都包含 'example.com' 或
# '*.example.com' 的主机头，并严格强制使用 https 访问。
middleware = [
    Middleware(
        TrustedHostMiddleware,
        allowed_hosts=['example.com', '*.example.com'],
    ),
    Middleware(HTTPSRedirectMiddleware)
]

app = Starlette(routes=routes, middleware=middleware)
```

每个 Starlette 应用程序默认自动包含两个中间件：

* `ServerErrorMiddleware` - 确保应用程序异常可以返回自定义的 500 页面，或者在调试模式下显示应用程序的堆栈跟踪。这是 *始终* 是最外层的中间件。
* `ExceptionMiddleware` - 添加异常处理程序，以便将特定类型的预期异常情况与处理函数关联。例如，在端点中抛出 `HTTPException(status_code=404)` 将呈现自定义的 404 页面。

中间件是从上到下进行评估的，因此我们示例应用程序中的执行流程将是：

* 中间件
    * `ServerErrorMiddleware`
    * `TrustedHostMiddleware`
    * `HTTPSRedirectMiddleware`
    * `ExceptionMiddleware`
* 路由
* 端点

以下是 Starlette 包中可用的中间件实现：

## CORSMiddleware

为外发响应添加适当的 [CORS 头](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)，以允许浏览器进行跨域请求。

CORSMiddleware 实现的默认参数通常是严格的，因此您需要明确启用特定的来源、方法或头部，才能允许浏览器在跨域上下文中使用它们。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.cors import CORSMiddleware

routes = ...

middleware = [
    Middleware(CORSMiddleware, allow_origins=['*'])
]

app = Starlette(routes=routes, middleware=middleware)
```

支持以下参数：

* `allow_origins` - 允许进行跨域请求的来源列表。例如 `['https://example.org', 'https://www.example.org']`。您可以使用 `['*']` 来允许任何来源。
* `allow_origin_regex` - 用于匹配应允许进行跨域请求的来源的正则表达式字符串。例如 `'https://.*\.example\.org'`。
* `allow_methods` - 应允许的跨域请求 HTTP 方法列表。默认为 `['GET']`。您可以使用 `['*']` 来允许所有标准方法。
* `allow_headers` - 应支持的跨域请求 HTTP 请求头列表。默认为 `[]`。您可以使用 `['*']` 来允许所有头部。`Accept`、`Accept-Language`、`Content-Language` 和 `Content-Type` 头部始终允许跨域请求。
* `allow_credentials` - 指示是否支持跨域请求中的 Cookies。默认为 `False`。另外，`allow_origins`、`allow_methods` 和 `allow_headers` 不能设置为 `['*']`，如果允许使用凭据，所有这些值必须明确指定。
* `expose_headers` - 指示应对浏览器开放的响应头部。默认为 `[]`。
* `max_age` - 设置浏览器缓存 CORS 响应的最大时间（以秒为单位）。默认为 `600`。

该中间件响应两种特定类型的 HTTP 请求...

#### CORS 预检请求

这些是任何带有 `Origin` 和 `Access-Control-Request-Method` 头部的 `OPTIONS` 请求。
在这种情况下，中间件将拦截传入请求，并使用适当的 CORS 头部进行响应，并根据需要返回 200 或 400 的响应作为信息。

#### 简单请求

任何带有 `Origin` 头部的请求。在这种情况下，中间件将正常传递请求，但会在响应中包含适当的 CORS 头部。

## SessionMiddleware

添加基于签名的 Cookie 的 HTTP 会话。会话信息是可读的，但不可修改。

您可以使用 `request.session` 字典接口访问或修改会话数据。

支持以下参数：

* `secret_key` - 应该是一个随机字符串。
* `session_cookie` - 默认为 "session"。
* `max_age` - 会话过期时间（以秒为单位）。默认为 2 周。如果设置为 `None`，则 Cookie 会在浏览器会话结束时过期。
* `same_site` - SameSite 标志防止浏览器与跨站请求一起发送会话 Cookie。默认为 `'lax'`。
* `path` - 为会话 Cookie 设置的路径。默认为 `'/'`。
* `https_only` - 指示应设置安全标志（仅可与 HTTPS 一起使用）。默认为 `False`。
* `domain` - 用于在子域或跨域之间共享 Cookie 的域。浏览器默认将域设置为设置 Cookie 的相同主机，排除子域（[参考](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#domain_attribute)）。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.sessions import SessionMiddleware

routes = ...

middleware = [
    Middleware(SessionMiddleware, secret_key=..., https_only=True)
]

app = Starlette(routes=routes, middleware=middleware)
```

## HTTPSRedirectMiddleware

强制所有传入的请求必须是 `https` 或 `wss`。任何传入的 `http` 或 `ws` 请求将被重定向到安全协议。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware

routes = ...

middleware = [
    Middleware(HTTPSRedirectMiddleware)
]

app = Starlette(routes=routes, middleware=middleware)
```

此中间件类没有配置选项。

## TrustedHostMiddleware

强制所有传入的请求都必须正确设置 `Host` 头部，以防止 HTTP 主机头攻击。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.trustedhost import TrustedHostMiddleware

routes = ...

middleware = [
    Middleware(TrustedHostMiddleware, allowed_hosts=['example.com', '*.example.com'])
]

app = Starlette(routes=routes, middleware=middleware)
```

支持以下参数：

* `allowed_hosts` - 应允许的主机名的域名列表。支持使用通配符域名，如 `*.example.com` 来匹配子域名。要允许任何主机名，可以使用 `allowed_hosts=["*"]` 或省略该中间件。
* `www_redirect` - 如果设置为 `True`，则请求到不带 `www` 的允许主机将被重定向到带 `www` 的对应主机。默认为 `True`。

如果传入的请求未正确验证，则会返回 400 响应。

## GZipMiddleware

处理任何在 `Accept-Encoding` 头部中包含 `"gzip"` 的请求的 GZip 响应。

该中间件将处理标准响应和流式响应。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.gzip import GZipMiddleware

routes = ...

middleware = [
    Middleware(GZipMiddleware, minimum_size=1000, compresslevel=9)
]

app = Starlette(routes=routes, middleware=middleware)
```

支持以下参数：

* `minimum_size` - 不对小于此最小字节数的响应进行 GZip 压缩。默认为 `500`。
* `compresslevel` - 在 GZip 压缩时使用的压缩级别。它是一个介于 1 到 9 之间的整数。默认为 `9`。较低的值会导致压缩速度更快，但文件较大，而较高的值会导致压缩较慢，但文件较小。

对于已经设置了 `Content-Encoding` 的响应，该中间件不会进行 GZip 压缩，以防止响应被压缩两次。

## BaseHTTPMiddleware

一个抽象类，允许您基于请求/响应接口编写 ASGI 中间件。

### 使用方法

要使用 `BaseHTTPMiddleware` 实现中间件类，必须重写 `async def dispatch(request, call_next)` 方法。

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.base import BaseHTTPMiddleware


class CustomHeaderMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers['Custom'] = 'Example'
        return response

routes = ...

middleware = [
    Middleware(CustomHeaderMiddleware)
]

app = Starlette(routes=routes, middleware=middleware)
```

如果您希望为中间件类提供配置选项，应该重写 `__init__` 方法，确保第一个参数是 `app`，其余的参数是可选的关键字参数。如果这样做，确保在实例中设置 `app` 属性。

```python
class CustomHeaderMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, header_value='Example'):
        super().__init__(app)
        self.header_value = header_value

    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers['Custom'] = self.header_value
        return response


middleware = [
    Middleware(CustomHeaderMiddleware, header_value='Customized')
]

app = Starlette(routes=routes, middleware=middleware)
```

中间件类不应在 `__init__` 方法外部修改其状态。相反，您应将任何状态保留在 `dispatch` 方法内，或者显式地传递它，而不是修改中间件实例。

### 限制

目前，`BaseHTTPMiddleware` 有一些已知的限制：

- 使用 `BaseHTTPMiddleware` 会阻止 [`contextlib.ContextVar`](https://docs.python.org/3/library/contextvars.html#contextvars.ContextVar) 的变化向上传播。也就是说，如果您在端点中为 `ContextVar` 设置了值，并尝试从中间件中读取它，您会发现读取的值与您在端点中设置的值不同（请参见 [此测试](https://github.com/encode/starlette/blob/621abc747a6604825190b93467918a0ec6456a24/tests/middleware/test_base.py#L192-L223) 了解此行为的示例）。

要克服这些限制，可以使用 [纯 ASGI 中间件](#pure-asgi-middleware)，如下所示。

## 纯 ASGI 中间件

[ASGI 规范](https://asgi.readthedocs.io/en/latest/) 使得能够直接使用 ASGI 接口实现 ASGI 中间件，作为一系列 ASGI 应用程序，彼此调用。事实上，Starlette 附带的中间件类就是通过这种方式实现的。

这种较低级的方法提供了更大的行为控制和增强的框架和服务器之间的互操作性。它还克服了 [`BaseHTTPMiddleware`](#limitations) 的限制。

### 编写纯 ASGI 中间件

创建 ASGI 中间件的最常见方法是使用类。

```python
class ASGIMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        await self.app(scope, receive, send)
```

上面的中间件是最基本的 ASGI 中间件。它接收一个父级 ASGI 应用程序作为构造函数的参数，并实现一个 `async __call__` 方法，调用该父应用程序。

一些实现，例如 [`asgi-cors`](https://github.com/simonw/asgi-cors/blob/10ef64bfcc6cd8d16f3014077f20a0fb8544ec39/asgi_cors.py)，使用了另一种风格，使用函数：

```python
import functools

def asgi_middleware():
    def asgi_decorator(app):

        @functools.wraps(app)
        async def wrapped_app(scope, receive, send):
            await app(scope, receive, send)

        return wrapped_app

    return asgi_decorator
```

无论哪种方式，ASGI 中间件必须是可调用的，接受三个参数：`scope`、`receive` 和 `send`。

* `scope` 是一个字典，包含关于连接的信息，其中 `scope["type"]` 可能是：
    * [`"http"`](https://asgi.readthedocs.io/en/latest/specs/www.html#http-connection-scope)：表示 HTTP 请求。
    * [`"websocket"`](https://asgi.readthedocs.io/en/latest/specs/www.html#websocket-connection-scope)：表示 WebSocket 连接。
    * [`"lifespan"`](https://asgi.readthedocs.io/en/latest/specs/lifespan.html#scope)：表示 ASGI 生命周期消息。
* `receive` 和 `send` 可以用于与 ASGI 服务器交换 ASGI 事件消息——更多内容见下文。消息的类型和内容取决于 `scope` 类型。了解更多内容，请参考 [ASGI 规范](https://asgi.readthedocs.io/en/latest/specs/index.html)。

### 使用纯 ASGI 中间件

纯 ASGI 中间件可以像其他中间件一样使用：

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware

from .middleware import ASGIMiddleware

routes = ...

middleware = [
    Middleware(ASGIMiddleware),
]

app = Starlette(..., middleware=middleware)
```

另见 [使用中间件](#using-middleware)。

### 类型注解

有两种方式为中间件添加类型注解：使用 Starlette 本身或 [`asgiref`](https://github.com/django/asgiref)。

* 使用 Starlette：适用于大多数常见用例。

```python
from starlette.types import ASGIApp, Message, Scope, Receive, Send


class ASGIMiddleware:
    def __init__(self, app: ASGIApp) -> None:
        self.app = app

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] != "http":
            return await self.app(scope, receive, send)

        async def send_wrapper(message: Message) -> None:
            # ... 执行某些操作
            await send(message)

        await self.app(scope, receive, send_wrapper)
```

* 使用 [`asgiref`](https://github.com/django/asgiref)：适用于更严格的类型提示。

```python
from asgiref.typing import ASGI3Application, ASGIReceiveCallable, ASGISendCallable, Scope
from asgiref.typing import ASGIReceiveEvent, ASGISendEvent


class ASGIMiddleware:
    def __init__(self, app: ASGI3Application) -> None:
        self.app = app

    async def __call__(self, scope: Scope, receive: ASGIReceiveCallable, send: ASGISendCallable) -> None:
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        async def send_wrapper(message: ASGISendEvent) -> None:
            # ... 执行某些操作
            await send(message)

        return await self.app(scope, receive, send_wrapper)
```

### 常见模式

#### 仅处理特定请求

ASGI 中间件可以根据 `scope` 的内容应用特定行为。

例如，仅处理 HTTP 请求，可以编写如下代码：

```python
class ASGIMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        ...  # 在此执行某些操作！

        await self.app(scope, receive, send)
```

同样，专门处理 WebSocket 的中间件可以通过 `scope["type"] != "websocket"` 来进行控制。

中间件还可以根据请求方法、URL、头部等条件表现不同的行为。

#### 重用 Starlette 组件

Starlette 提供了多个数据结构，接受 ASGI 的 `scope`、`receive` 和/或 `send` 参数，使你能够在更高抽象层次上进行工作。这些数据结构包括 [`Request`](requests.md#request)、[`Headers`](requests.md#headers)、[`QueryParams`](requests.md#query-parameters)、[`URL`](requests.md#url) 等。

例如，你可以实例化一个 `Request` 来更轻松地检查 HTTP 请求：

```python
from starlette.requests import Request

class ASGIMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        if scope["type"] == "http":
            request = Request(scope)
            ... # 使用 `request.method`、`request.url`、`request.headers` 等。

        await self.app(scope, receive, send)
```

你还可以重用 [responses](responses.md)，它们也是 ASGI 应用程序。

#### 发送主动响应

检查连接的 `scope` 可以使你有条件地调用另一个 ASGI 应用程序。一个用例可能是在不调用应用程序的情况下直接发送响应。

例如，这个中间件使用字典来根据请求路径执行永久重定向。这可以用来在需要重构路由 URL 模式时，继续支持旧的 URL。

```python
from starlette.datastructures import URL
from starlette.responses import RedirectResponse

class RedirectsMiddleware:
    def __init__(self, app, path_mapping: dict):
        self.app = app
        self.path_mapping = path_mapping

    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        url = URL(scope=scope)

        if url.path in self.path_mapping:
            url = url.replace(path=self.path_mapping[url.path])
            response = RedirectResponse(url, status_code=301)
            await response(scope, receive, send)
            return

        await self.app(scope, receive, send)
```

示例用法如下：

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware

routes = ...

redirections = {
    "/v1/resource/": "/v2/resource/",
    # ...
}

middleware = [
    Middleware(RedirectsMiddleware, path_mapping=redirections),
]

app = Starlette(routes=routes, middleware=middleware)
```

#### 检查或修改请求

可以通过操作 `scope` 来访问或修改请求信息。要查看此模式的完整示例，请参见 Uvicorn 的 [`ProxyHeadersMiddleware`](https://github.com/encode/uvicorn/blob/fd4386fefb8fe8a4568831a7d8b2930d5fb61455/uvicorn/middleware/proxy_headers.py)，它在代理前端后为请求检查和调整 `scope`。

此外，包装 `receive` ASGI 可调用函数允许你通过操作 [`http.request`](https://asgi.readthedocs.io/en/latest/specs/www.html#request-receive-event) ASGI 事件消息来访问或修改 HTTP 请求体。

例如，这个中间件计算并记录传入请求体的大小：

```python
class LoggedRequestBodySizeMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        body_size = 0

        async def receive_logging_request_body_size():
            nonlocal body_size

            message = await receive()
            assert message["type"] == "http.request"

            body_size += len(message.get("body", b""))

            if not message.get("more_body", False):
                print(f"请求体的大小是：{body_size} 字节")

            return message

        await self.app(scope, receive_logging_request_body_size, send)
```

同样，WebSocket 中间件可以操作 [`websocket.receive`](https://asgi.readthedocs.io/en/latest/specs/www.html#receive-receive-event) ASGI 事件消息来检查或更改传入的 WebSocket 数据。

有关更改 HTTP 请求体的示例，请参见 [`msgpack-asgi`](https://github.com/florimondmanca/msgpack-asgi)。

#### 检查或修改响应

包装 `send` ASGI 可调用函数可以让你检查或修改底层应用程序发送的 HTTP 响应。为此，你可以对 [`http.response.start`](https://asgi.readthedocs.io/en/latest/specs/www.html#response-start-send-event) 或 [`http.response.body`](https://asgi.readthedocs.io/en/latest/specs/www.html#response-body-send-event) ASGI 事件消息作出反应。

例如，这个中间件会添加一些固定的额外响应头：

```python
from starlette.datastructures import MutableHeaders

class ExtraResponseHeadersMiddleware:
    def __init__(self, app, headers):
        self.app = app
        self.headers = headers

    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            return await self.app(scope, receive, send)

        async def send_with_extra_headers(message):
            if message["type"] == "http.response.start":
                headers = MutableHeaders(scope=message)
                for key, value in self.headers:
                    headers.append(key, value)

            await send(message)

        await self.app(scope, receive, send_with_extra_headers)
```

另请参见 [`asgi-logger`](https://github.com/Kludex/asgi-logger/blob/main/asgi_logger/middleware.py)，它是一个检查 HTTP 响应并记录可配置的 HTTP 访问日志行的示例。

同样，WebSocket 中间件可以操作 [`websocket.send`](https://asgi.readthedocs.io/en/latest/specs/www.html#send-send-event) ASGI 事件消息来检查或修改发出的 WebSocket 数据。

请注意，如果你更改了响应体，必须更新响应的 `Content-Length` 头以匹配新的响应体长度。完整示例请参见 [`brotli-asgi`](https://github.com/fullonic/brotli-asgi)。

#### 将信息传递给端点

如果需要与底层应用程序或端点共享信息，可以将其存储在 `scope` 字典中。请注意，这只是一种约定——例如，Starlette 使用它来与端点共享路由信息——但它并不是 ASGI 规范的一部分。如果这样做，务必通过使用低概率冲突的键来避免与其他中间件或应用程序发生冲突。

例如，当包含以下中间件时，端点将能够访问 `request.scope["asgi_transaction_id"]`。

```python
import uuid

class TransactionIDMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        scope["asgi_transaction_id"] = uuid.uuid4()
        await self.app(scope, receive, send)
```

#### 清理和错误处理

你可以将应用程序包装在 `try/except/finally` 块中或使用上下文管理器来执行清理操作或处理错误。

例如，以下中间件可能会收集度量数据并处理应用程序异常...

```python
import time

class MonitoringMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        start = time.time()
        try:
            await self.app(scope, receive, send)
        except Exception as exc:
            ...  # 处理异常
            raise
        finally:
            end = time.time()
            elapsed = end - start
            ...  # 将 `elapsed` 作为度量提交到监控后端
```

另请参见 [`timing-asgi`](https://github.com/steinnes/timing-asgi)，它是这种模式的完整示例。

### 注意事项

#### ASGI 中间件应为无状态

由于 ASGI 设计用于处理并发请求，任何特定于连接的状态应仅限于 `__call__` 实现。否则，通常会导致跨请求的变量读写冲突，并可能引发 bug。

例如，以下代码会在响应中存在 `X-Mock` 头时有条件地替换响应体：

=== "✅ 做法"
    
    ```python
    from starlette.datastructures import Headers
    
    class MockResponseBodyMiddleware:
        def __init__(self, app, content):
            self.app = app
            self.content = content
    
        async def __call__(self, scope, receive, send):
            if scope["type"] != "http":
                await self.app(scope, receive, send)
                return
    
            # 如果 HTTP 响应包含 'X-Mock' 头，则将标志设为 True。
            # ✅: 作用域仅限于此函数。
            should_mock = False
    
            async def maybe_send_with_mock_content(message):
                nonlocal should_mock
    
                if message["type"] == "http.response.start":
                    headers = Headers(raw=message["headers"])
                    should_mock = headers.get("X-Mock") == "1"
                    await send(message)
    
                elif message["type"] == "http.response.body":
                    if should_mock:
                        message = {"type": "http.response.body", "body": self.content}
                    await send(message)
    
            await self.app(scope, receive, maybe_send_with_mock_content)
    ```

=== "❌ 不要做"

    ```python hl_lines="7-8"
    from starlette.datastructures import Headers
    
    class MockResponseBodyMiddleware:
        def __init__(self, app, content):
            self.app = app
            self.content = content
            # ❌: 这个变量会跨请求被读写！
            self.should_mock = False
    
        async def __call__(self, scope, receive, send):
            if scope["type"] != "http":
                await self.app(scope, receive, send)
                return
    
            async def maybe_send_with_mock_content(message):
                if message["type"] == "http.response.start":
                    headers = Headers(raw=message["headers"])
                    self.should_mock = headers.get("X-Mock") == "1"
                    await send(message)
    
                elif message["type"] == "http.response.body":
                    if self.should_mock:
                        message = {"type": "http.response.body", "body": self.content}
                    await send(message)
    
            await self.app(scope, receive, maybe_send_with_mock_content)
    ```

另见 [`GZipMiddleware`](https://github.com/encode/starlette/blob/9ef1b91c9c043197da6c3f38aa153fd874b95527/starlette/middleware/gzip.py)，它是一个完整的示例，演示如何避免这一潜在的陷阱。

### 深入阅读

本文档应该能为你提供如何创建 ASGI 中间件的良好基础。

不过，关于该主题也有很多精彩的文章：

- [ASGI 介绍：异步 Python Web 生态系统的崛起](https://florimond.dev/en/posts/2019/08/introduction-to-asgi-async-python-web/)
- [如何编写 ASGI 中间件](https://pgjones.dev/blog/how-to-write-asgi-middleware-2021/)

## 在其他框架中使用中间件

为了在其他 ASGI 应用程序上包装 ASGI 中间件，应该使用更通用的模式来包装应用程序实例：

```python
app = TrustedHostMiddleware(app, allowed_hosts=['example.com'])
```

你也可以在 Starlette 应用程序实例上使用这种方式，但更推荐使用 `middleware=<中间件实例列表>` 方式，因为它能：

* 确保所有内容都被包装在一个最外层的 `ServerErrorMiddleware` 中。
* 保留顶级的 `app` 实例。

## 将中间件应用于路由组

中间件还可以添加到 `Mount` 实例中，这样你可以将中间件应用于一组路由或子应用程序：

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.gzip import GZipMiddleware
from starlette.routing import Mount, Route


routes = [
    Mount(
        "/",
        routes=[
            Route(
                "/example",
                endpoint=...,
            )
        ],
        middleware=[Middleware(GZipMiddleware)]
    )
]

app = Starlette(routes=routes)
```

请注意，以这种方式使用的中间件 *不会* 包装在异常处理中间件中，像应用于 `Starlette` 应用程序的中间件那样。这通常不是问题，因为它只适用于检查或修改 `Response` 的中间件，而且即使是这样，你也可能不希望将此逻辑应用于错误响应。如果你确实只希望将中间件逻辑应用于某些路由的错误响应，可以选择以下几种方法：

* 在 `Mount` 上添加 `ExceptionMiddleware`
* 在中间件中添加 `try/except` 块，并从那里返回错误响应
* 将标记和处理分成两个中间件，一个放在 `Mount` 上，用于标记响应需要处理（例如，通过设置 `scope["log-response"] = True`），另一个应用于 `Starlette` 应用程序，负责实际处理

`Route`/`WebSocket` 类也接受 `middleware` 参数，这允许你将中间件应用于单个路由：

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.gzip import GZipMiddleware
from starlette.routing import Route


routes = [
    Route(
        "/example",
        endpoint=...,
        middleware=[Middleware(GZipMiddleware)]
    )
]

app = Starlette(routes=routes)
```

你还可以将中间件应用于 `Router` 类，这允许你将中间件应用于一组路由：

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.gzip import GZipMiddleware
from starlette.routing import Route, Router


routes = [
    Route("/example", endpoint=...),
    Route("/another", endpoint=...),
]

router = Router(routes=routes, middleware=[Middleware(GZipMiddleware)])
```

## 第三方中间件

#### [asgi-auth-github](https://github.com/simonw/asgi-auth-github)

该中间件为任何 ASGI 应用程序添加身份验证，要求用户使用其 GitHub 账户进行登录（通过 [OAuth](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/)）。可以将访问权限限制为特定用户或特定 GitHub 组织或团队的成员。

#### [asgi-csrf](https://github.com/simonw/asgi-csrf)

用于防止 CSRF 攻击的中间件。该中间件实现了双重提交 Cookie 模式，其中设置了一个 Cookie，然后将其与 csrftoken 隐藏表单字段或 `x-csrftoken` HTTP 头进行比较。

#### [AuthlibMiddleware](https://github.com/aogier/starlette-authlib)

作为 Starlette 会话中间件的替代，使用 [authlib 的 jwt](https://docs.authlib.org/en/latest/jose/jwt.html) 模块。

#### [BugsnagMiddleware](https://github.com/ashinabraham/starlette-bugsnag)

一个用于将异常记录到 [Bugsnag](https://www.bugsnag.com/) 的中间件类。

#### [CSRFMiddleware](https://github.com/frankie567/starlette-csrf)

用于防止 CSRF 攻击的中间件。该中间件实现了双重提交 Cookie 模式，其中设置了一个 Cookie，然后将其与 `x-csrftoken` HTTP 头进行比较。

#### [EarlyDataMiddleware](https://github.com/HarrySky/starlette-early-data)

用于检测和拒绝 [TLSv1.3 早期数据](https://tools.ietf.org/html/rfc8470) 请求的中间件和装饰器。

#### [PrometheusMiddleware](https://github.com/perdy/starlette-prometheus)

一个用于捕获与请求和响应相关的 Prometheus 指标的中间件类，包括正在进行的请求、计时等。

#### [ProxyHeadersMiddleware](https://github.com/encode/uvicorn/blob/master/uvicorn/middleware/proxy_headers.py)

Uvicorn 包含一个中间件类，用于在使用代理服务器时根据 `X-Forwarded-Proto` 和 `X-Forwarded-For` 头部确定客户端 IP 地址。对于更复杂的代理配置，可能需要调整此中间件。

#### [RateLimitMiddleware](https://github.com/abersheeran/asgi-ratelimit)

一个速率限制中间件。使用正则表达式匹配 URL；灵活的规则；高度可定制。非常易于使用。

#### [RequestIdMiddleware](https://github.com/snok/asgi-correlation-id)

一个用于读取/生成请求 ID 并将其附加到应用程序日志的中间件类。

#### [RollbarMiddleware](https://docs.rollbar.com/docs/starlette)

一个用于将异常、错误和日志消息记录到 [Rollbar](https://www.rollbar.com) 的中间件类。

#### [StarletteOpentracing](https://github.com/acidjunk/starlette-opentracing)

一个将追踪信息发射到 [OpenTracing.io](https://opentracing.io/) 兼容追踪器的中间件类，可用于分析和监控分布式应用程序。

#### [SecureCookiesMiddleware](https://github.com/thearchitector/starlette-securecookies)

一个可定制的中间件，用于为 Starlette 应用程序添加自动的 Cookie 加密和解密功能，并额外支持现有的基于 Cookie 的中间件。

#### [TimingMiddleware](https://github.com/steinnes/timing-asgi)

一个中间件类，用于为每个通过它的请求发射时间信息（CPU 和壁钟时间）。包括如何将这些时间作为 statsd 指标发射的示例。

#### [WSGIMiddleware](https://github.com/abersheeran/a2wsgi)

一个中间件类，用于将 WSGI 应用程序转换为 ASGI 应用程序。
