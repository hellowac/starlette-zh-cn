## HTTP 路由

Starlette 提供了一个简单但功能强大的请求路由系统。路由表被定义为一个路由列表，并在实例化应用时传入。

```python
from starlette.applications import Starlette
from starlette.responses import PlainTextResponse
from starlette.routing import Route


async def homepage(request):
    return PlainTextResponse("Homepage")

async def about(request):
    return PlainTextResponse("About")


routes = [
    Route("/", endpoint=homepage),
    Route("/about", endpoint=about),
]

app = Starlette(routes=routes)
```

`endpoint` 参数可以是以下之一：

* 一个普通函数或异步函数，接受一个 `request` 参数，并返回一个响应。
* 一个实现了 ASGI 接口的类，例如 Starlette 的 [HTTPEndpoint](endpoints.md#httpendpoint)。

## 路径参数

路径可以使用 URI 模板风格来捕获路径组件。

```python
Route('/users/{username}', user)
```
默认情况下，这将捕获路径结束或下一个 `/` 之前的所有字符。

你可以使用转换器来修改捕获内容。可用的转换器包括：

* `str` 返回一个字符串，这是默认的转换器。
* `int` 返回一个 Python 整数。
* `float` 返回一个 Python 浮点数。
* `uuid` 返回一个 Python `uuid.UUID` 实例。
* `path` 返回剩余的路径，包括任何额外的 `/` 字符。

转换器通过在路径中使用冒号前缀来使用，例如：

```python
Route('/users/{user_id:int}', user)
Route('/floating-point/{number:float}', floating_point)
Route('/uploaded/{rest_of_path:path}', uploaded)
```

如果你需要一个未定义的不同转换器，可以创建自己的转换器。以下是如何创建一个 `datetime` 转换器并注册它的示例：

```python
from datetime import datetime

from starlette.convertors import Convertor, register_url_convertor


class DateTimeConvertor(Convertor):
    regex = "[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(.[0-9]+)?"

    def convert(self, value: str) -> datetime:
        return datetime.strptime(value, "%Y-%m-%dT%H:%M:%S")

    def to_string(self, value: datetime) -> str:
        return value.strftime("%Y-%m-%dT%H:%M:%S")

register_url_convertor("datetime", DateTimeConvertor())
```

注册后，你将能够像这样使用它：

```python
Route('/history/{date:datetime}', history)
```

路径参数通过 `request.path_params` 字典提供。

```python
async def user(request):
    user_id = request.path_params['user_id']
    ...
```

## 处理 HTTP 方法

路由还可以指定一个端点处理哪些 HTTP 方法：

```python
Route('/users/{user_id:int}', user, methods=["GET", "POST"])
```

默认情况下，函数端点只会接受 `GET` 请求，除非另有指定。

## 子路由挂载

在大型应用中，你可能希望根据共同的路径前缀将路由表的部分拆分出来。

```python
routes = [
    Route('/', homepage),
    Mount('/users', routes=[
        Route('/', users, methods=['GET', 'POST']),
        Route('/{username}', user),
    ])
]
```

这种风格允许你在项目的不同部分定义路由表的不同子集。

```python
from myproject import users, auth

routes = [
    Route('/', homepage),
    Mount('/users', routes=users.routes),
    Mount('/auth', routes=auth.routes),
]
```

你还可以使用挂载将子应用包含在 Starlette 应用中。例如...

```python
# 这是一个独立的静态文件服务器：
app = StaticFiles(directory="static")

# 这是一个在 Starlette 应用内挂载的静态文件服务器，
# 位于 "/static" 路径下。
routes = [
    ...
    Mount("/static", app=StaticFiles(directory="static"), name="static")
]

app = Starlette(routes=routes)
```

## 反向 URL 查找

你通常希望能够为特定路由生成 URL，特别是在需要返回重定向响应时。

* 签名：`url_for(name, **path_params) -> URL`

```python
routes = [
    Route("/", homepage, name="homepage")
]

# 我们可以使用以下代码来返回 URL...
url = request.url_for("homepage")
```

URL 查找可以包括路径参数...

```python
routes = [
    Route("/users/{username}", user, name="user_detail")
]

# 我们可以使用以下代码来返回 URL...
url = request.url_for("user_detail", username=...)
```

如果一个 `Mount` 包含 `name`，则子路由应使用 `{prefix}:{name}` 风格进行反向 URL 查找。

```python
routes = [
    Mount("/users", name="users", routes=[
        Route("/", user, name="user_list"),
        Route("/{username}", user, name="user_detail")
    ])
]

# 我们可以使用以下代码来返回 URL...
url = request.url_for("users:user_list")
url = request.url_for("users:user_detail", username=...)
```

挂载的应用可以包括 `path=...` 参数。

```python
routes = [
    ...
    Mount("/static", app=StaticFiles(directory="static"), name="static")
]

# 我们可以使用以下代码来返回 URL...
url = request.url_for("static", path="/css/base.css")
```

对于没有 `request` 实例的情况，你可以对应用程序进行反向查找，尽管这些只会返回 URL 路径。

```python
url = app.url_path_for("user_detail", username=...)
```

## 基于主机的路由

如果你希望根据 `Host` 头部使用不同的路由来处理相同的路径，可以使用基于主机的路由。

请注意，匹配时会从 `Host` 头部移除端口。例如，`Host (host='example.org:3600', ...)` 将被处理，即使 `Host` 头部包含或不包含除 `3600` 外的端口（例如 `example.org:5600`、`example.org`）。因此，如果需要在 `url_for` 中使用端口，可以在主机名中指定端口。

有几种方法可以将基于主机的路由连接到应用程序中：

```python
site = Router()  # 例如使用 `@site.route()` 来配置。
api = Router()  # 例如使用 `@api.route()` 来配置。
news = Router()  # 例如使用 `@news.route()` 来配置。

routes = [
    Host('api.example.org', api, name="site_api")
]

app = Starlette(routes=routes)

app.host('www.example.org', site, name="main_site")

news_host = Host('news.example.org', news)
app.router.routes.append(news_host)
```

URL 查找可以像路径参数一样包括主机参数：

```python
routes = [
    Host("{subdomain}.example.org", name="sub", app=Router(routes=[
        Mount("/users", name="users", routes=[
            Route("/", user, name="user_list"),
            Route("/{username}", user, name="user_detail")
        ])
    ]))
]
...
url = request.url_for("sub:users:user_detail", username=..., subdomain=...)
url = request.url_for("sub:users:user_list", subdomain=...)
```

## 路由优先级

传入路径会依次与每个 `Route` 进行匹配。

如果多个路由都可能匹配传入路径，应该确保更具体的路由排在一般情况之前。

例如：

```python
# 不要这样做：`/users/me` 永远不会匹配传入请求。
routes = [
    Route('/users/{username}', user),
    Route('/users/me', current_user),
]

# 这样做：`/users/me` 会首先被测试。
routes = [
    Route('/users/me', current_user),
    Route('/users/{username}', user),
]
```

## 使用 Router 实例

如果你在底层工作，可能希望使用一个普通的 `Router` 实例，而不是创建一个 `Starlette` 应用。这将为你提供一个轻量级的 ASGI 应用，只提供应用路由，而不包含任何中间件。

```python
app = Router(routes=[
    Route('/', homepage),
    Mount('/users', routes=[
        Route('/', users, methods=['GET', 'POST']),
        Route('/{username}', user),
    ])
])
```

## WebSocket 路由

在处理 WebSocket 端点时，应该使用 `WebSocketRoute`，而不是通常的 `Route`。

路径参数和 `WebSocketRoute` 的反向 URL 查找与 HTTP `Route` 相同，可以在 HTTP [路由](#http)部分找到相关说明。

```python
from starlette.applications import Starlette
from starlette.routing import WebSocketRoute


async def websocket_index(websocket):
    await websocket.accept()
    await websocket.send_text("Hello, websocket!")
    await websocket.close()


async def websocket_user(websocket):
    name = websocket.path_params["name"]
    await websocket.accept()
    await websocket.send_text(f"Hello, {name}")
    await websocket.close()


routes = [
    WebSocketRoute("/", endpoint=websocket_index),
    WebSocketRoute("/{name}", endpoint=websocket_user),
]

app = Starlette(routes=routes)
```

`endpoint` 参数可以是以下之一：

* 一个异步函数，接受一个 `websocket` 参数。
* 一个实现 ASGI 接口的类，例如 Starlette 的 [WebSocketEndpoint](endpoints.md#websocketendpoint)。
