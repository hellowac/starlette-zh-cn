
Starlette 允许您安装自定义异常处理器来处理发生错误或已处理异常时返回响应的方式。

```python
from starlette.applications import Starlette
from starlette.exceptions import HTTPException
from starlette.requests import Request
from starlette.responses import HTMLResponse


HTML_404_PAGE = ...
HTML_500_PAGE = ...


async def not_found(request: Request, exc: HTTPException):
    return HTMLResponse(content=HTML_404_PAGE, status_code=exc.status_code)

async def server_error(request: Request, exc: HTTPException):
    return HTMLResponse(content=HTML_500_PAGE, status_code=exc.status_code)


exception_handlers = {
    404: not_found,
    500: server_error
}

app = Starlette(routes=routes, exception_handlers=exception_handlers)
```

如果启用了 `debug` 模式并发生错误，则 Starlette 会响应回溯信息，而不是使用已安装的 500 错误处理器。

```python
app = Starlette(debug=True, routes=routes, exception_handlers=exception_handlers)
```

除了为特定的状态码注册处理器外，您还可以为异常类别注册处理器。

特别是，您可能希望重写如何处理内置的 `HTTPException` 类。例如，使用 JSON 样式的响应：

```python
async def http_exception(request: Request, exc: HTTPException):
    return JSONResponse({"detail": exc.detail}, status_code=exc.status_code)

exception_handlers = {
    HTTPException: http_exception
}
```

`HTTPException` 还带有 `headers` 参数，允许将头信息传递给响应类：

```python
async def http_exception(request: Request, exc: HTTPException):
    return JSONResponse(
        {"detail": exc.detail},
        status_code=exc.status_code,
        headers=exc.headers
    )
```

您还可能想要重写 `WebSocketException` 的处理方式：

```python
async def websocket_exception(websocket: WebSocket, exc: WebSocketException):
    await websocket.close(code=1008)

exception_handlers = {
    WebSocketException: websocket_exception
}
```

## 错误和已处理异常

区分已处理异常和错误是非常重要的。

已处理异常并不表示错误情况。它们会被转换为适当的 HTTP 响应，并通过标准的中间件栈发送。默认情况下，`HTTPException` 类用于管理所有已处理的异常。

错误是应用程序中发生的其他任何异常。这些异常应该沿着整个中间件栈传播作为异常。任何错误日志中间件应确保将异常重新抛出，直到服务器。

在实际操作中，错误处理器使用的是 `exception_handler[500]` 或 `exception_handler[Exception]`。这两个键都可以使用，见下例：

```python
async def handle_error(request: Request, exc: HTTPException):
    # 执行一些逻辑
    return JSONResponse({"detail": exc.detail}, status_code=exc.status_code)

exception_handlers = {
    Exception: handle_error  # 或 "500: handle_error"
}
```

需要注意的是，如果 [`BackgroundTask`](https://www.starlette.io/background/) 引发异常，它将由 `handle_error` 函数处理，但此时响应已经发送。换句话说，`handle_error` 创建的响应将被丢弃。如果错误发生在响应发送之前，它将使用响应对象——在上述示例中返回的 `JSONResponse`。

为了正确处理这种行为，`Starlette` 应用程序的中间件栈配置如下：

* `ServerErrorMiddleware` - 当发生服务器错误时返回 500 响应。
* 安装的中间件(Installed middleware)
* `ExceptionMiddleware` - 处理已处理的异常并返回响应。
* 路由器(Router)
* 端点(Endpoints)

## HTTPException

`HTTPException` 类提供了一个基础类，您可以用于处理任何已处理的异常。`ExceptionMiddleware` 实现默认将为任何 `HTTPException` 返回纯文本 HTTP 响应。

* `HTTPException(status_code, detail=None, headers=None)`

您应该只在路由或端点中引发 `HTTPException`。中间件类应直接返回适当的响应，而不是引发异常。

如果在 `websocket.accept()` 之前引发 `HTTPException`，则可以在 WebSocket 端点中使用该异常。连接不会升级为 WebSocket 连接，而是返回适当的 HTTP 响应。

```python
from starlette.applications import Starlette
from starlette.exceptions import HTTPException
from starlette.routing import WebSocketRoute
from starlette.websockets import WebSocket


async def websocket_endpoint(websocket: WebSocket):
    raise HTTPException(status_code=400, detail="Bad request")


app = Starlette(routes=[WebSocketRoute("/ws", websocket_endpoint)])
```

## WebSocketException

您可以使用 `WebSocketException` 类在 WebSocket 端点中引发错误。

* `WebSocketException(code=1008, reason=None)`

您可以设置任何有效的代码，具体请参见 [规范](https://tools.ietf.org/html/rfc6455#section-7.4.1) 中的定义。
