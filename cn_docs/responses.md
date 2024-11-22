Starlette 包含了一些响应类，用于通过 `send` 通道发送适当的 ASGI 消息。

### 响应（Response）

签名：`Response(content, status_code=200, headers=None, media_type=None)`

- **`content`**：字符串或字节串。
- **`status_code`**：整数类型的 HTTP 状态码。
- **`headers`**：字符串字典，用于定义 HTTP 头信息。
- **`media_type`**：字符串，指定媒体类型，例如 `"text/html"`。

Starlette 会自动添加 `Content-Length` 头信息，还会根据 `media_type` 自动设置 `Content-Type` 头信息。如果是文本类型，将附加字符集（charset），除非 `media_type` 已明确指定了字符集。

创建响应后，可以将其作为 ASGI 应用实例发送：

```python
from starlette.responses import Response


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = Response('Hello, world!', media_type='text/plain')
    await response(scope, receive, send)
```

#### 设置 Cookie

Starlette 提供了 `set_cookie` 方法，用于在响应对象中设置 Cookie。

签名：`Response.set_cookie(key, value, max_age=None, expires=None, path="/", domain=None, secure=False, httponly=False, samesite="lax")`

- **`key`**：字符串，表示 Cookie 的键。
- **`value`**：字符串，表示 Cookie 的值。
- **`max_age`**：整数，定义 Cookie 的生命周期（秒）。负值或 `0` 会立即丢弃 Cookie。可选。
- **`expires`**：整数（以秒为单位）或 `datetime` 对象，定义 Cookie 的到期时间。可选。
- **`path`**：字符串，指定 Cookie 应用的路径范围。可选。
- **`domain`**：字符串，指定 Cookie 有效的域。可选。
- **`secure`**：布尔值，指定 Cookie 是否仅在使用 SSL 和 HTTPS 协议时发送。可选。
- **`httponly`**：布尔值，指定 Cookie 是否不能通过 JavaScript（如 `Document.cookie`）访问。可选。
- **`samesite`**：字符串，指定 Cookie 的 SameSite 策略。有效值为 `'lax'`、`'strict'` 和 `'none'`，默认值为 `'lax'`。可选。

#### 删除 Cookie

Starlette 还提供了 `delete_cookie` 方法，用于手动使已设置的 Cookie 失效。

签名：`Response.delete_cookie(key, path='/', domain=None)`

### HTML 响应（HTMLResponse）

接收文本或字节并返回一个 HTML 响应：

```python
from starlette.responses import HTMLResponse


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = HTMLResponse('<html><body><h1>Hello, world!</h1></body></html>')
    await response(scope, receive, send)
```

### 文本响应（PlainTextResponse）

接收一些文本或字节并返回纯文本响应。

```python
from starlette.responses import PlainTextResponse


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = PlainTextResponse('Hello, world!')
    await response(scope, receive, send)
```

### JSON 响应（JSONResponse）

接收一些数据并返回 `application/json` 编码的响应。

```python
from starlette.responses import JSONResponse


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = JSONResponse({'hello': 'world'})
    await response(scope, receive, send)
```

#### 自定义 JSON 序列化

如果需要对 JSON 序列化进行细粒度控制，可以通过继承 `JSONResponse` 并重写 `render` 方法实现。

例如，若要使用第三方 JSON 库 [orjson](https://pypi.org/project/orjson/)：

```python
from typing import Any

import orjson
from starlette.responses import JSONResponse


class OrjsonResponse(JSONResponse):
    def render(self, content: Any) -> bytes:
        return orjson.dumps(content)
```

通常情况下，默认使用 `JSONResponse` 即可，除非需要对特定端点进行微优化，或需要序列化非标准对象类型。

### 重定向响应（RedirectResponse）

返回 HTTP 重定向响应。默认使用状态码 307。

```python
from starlette.responses import PlainTextResponse, RedirectResponse


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    if scope['path'] != '/':
        response = RedirectResponse(url='/')
    else:
        response = PlainTextResponse('Hello, world!')
    await response(scope, receive, send)
```

### 流式响应（StreamingResponse）

接收一个异步生成器或普通生成器/迭代器，并以流的方式返回响应体。

```python
from starlette.responses import StreamingResponse
import asyncio


async def slow_numbers(minimum, maximum):
    yield '<html><body><ul>'
    for number in range(minimum, maximum + 1):
        yield '<li>%d</li>' % number
        await asyncio.sleep(0.5)
    yield '</ul></body></html>'


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    generator = slow_numbers(1, 10)
    response = StreamingResponse(generator, media_type='text/html')
    await response(scope, receive, send)
```

请注意，<a href="https://docs.python.org/3/glossary.html#term-file-like-object" target="_blank">类文件对象</a>（如通过 `open()` 创建的文件对象）是普通迭代器，因此可以直接作为 `StreamingResponse` 返回。

---

### 文件响应（FileResponse）

以异步流的方式发送文件作为响应。

与其他响应类型不同，其实例化时接受一组不同的参数：

- **`path`**：要发送的文件路径。
- **`headers`**：包含自定义头的字典。
- **`media_type`**：字符串类型，表示媒体类型。如果未设置，将根据文件名或路径推断媒体类型。
- **`filename`**：如果设置，将在响应的 `Content-Disposition` 中包含该值。
- **`content_disposition_type`**：指定 `Content-Disposition` 的类型，可以设置为 `"attachment"`（默认）或 `"inline"`。

文件响应将自动包含适当的 `Content-Length`、`Last-Modified` 和 `ETag` 头。

```python
from starlette.responses import FileResponse


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = FileResponse('statics/favicon.ico')
    await response(scope, receive, send)
```

文件响应还支持 [HTTP 范围请求](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests)。

- 如果文件存在，响应中将包含 `Accept-Ranges: bytes` 头，目前仅支持 `bytes` 单位。
- 如果请求中包含 `Range` 头且文件存在，将返回 `206 Partial Content` 响应，并包含请求的字节范围。
- 如果范围无效，将返回 `416 Range Not Satisfiable` 响应。

### 第三方响应

#### [EventSourceResponse](https://github.com/sysid/sse-starlette)

一个实现 [服务器发送事件（Server-Sent Events）](https://html.spec.whatwg.org/multipage/server-sent-events.html) 的响应类。它可以在不使用 WebSocket 的情况下，实现从服务器到客户端的事件流式传输。
