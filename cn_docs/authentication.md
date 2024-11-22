Starlette 提供了一个简单但强大的接口，用于处理身份验证和权限控制。安装了适当的身份验证后端的 `AuthenticationMiddleware` 后，`request.user` 和 `request.auth` 接口将在您的端点中可用。

```python
from starlette.applications import Starlette
from starlette.authentication import (
    AuthCredentials, AuthenticationBackend, AuthenticationError, SimpleUser
)
from starlette.middleware import Middleware
from starlette.middleware.authentication import AuthenticationMiddleware
from starlette.responses import PlainTextResponse
from starlette.routing import Route
import base64
import binascii


class BasicAuthBackend(AuthenticationBackend):
    async def authenticate(self, conn):
        if "Authorization" not in conn.headers:
            return

        auth = conn.headers["Authorization"]
        try:
            scheme, credentials = auth.split()
            if scheme.lower() != 'basic':
                return
            decoded = base64.b64decode(credentials).decode("ascii")
        except (ValueError, UnicodeDecodeError, binascii.Error) as exc:
            raise AuthenticationError('Invalid basic auth credentials')

        username, _, password = decoded.partition(":")
        # TODO: 在这里验证用户名和密码
        return AuthCredentials(["authenticated"]), SimpleUser(username)


async def homepage(request):
    if request.user.is_authenticated:
        return PlainTextResponse('Hello, ' + request.user.display_name)
    return PlainTextResponse('Hello, you')

routes = [
    Route("/", endpoint=homepage)
]

middleware = [
    Middleware(AuthenticationMiddleware, backend=BasicAuthBackend())
]

app = Starlette(routes=routes, middleware=middleware)
```

## 用户

安装了 `AuthenticationMiddleware` 后，`request.user` 接口将可用于端点或其他中间件。

该接口应继承自 `BaseUser`，提供两个属性，以及用户模型包含的其他信息：

* `.is_authenticated`
* `.display_name`

Starlette 提供了两种内置的用户实现：`UnauthenticatedUser()` 和 `SimpleUser(username)`。

## AuthCredentials

重要的是，身份验证凭据应与用户作为独立的概念进行处理。身份验证方案应该能够独立于用户身份，限制或授予特定的权限。

`AuthCredentials` 类提供了 `request.auth` 暴露的基本接口：

* `.scopes`

## 权限

权限通过端点装饰器实现，该装饰器强制要求传入的请求包含所需的身份验证作用域。

```python
from starlette.authentication import requires


@requires('authenticated')
async def dashboard(request):
    ...
```

您可以包含一个或多个所需的作用域：

```python
from starlette.authentication import requires


@requires(['authenticated', 'admin'])
async def dashboard(request):
    ...
```

默认情况下，当权限未被授予时，将返回 403 响应。在某些情况下，您可能希望自定义这一行为，例如，向未认证的用户隐藏有关 URL 布局的信息。

```python
from starlette.authentication import requires


@requires(['authenticated', 'admin'], status_code=404)
async def dashboard(request):
    ...
```

!!! 注意

    `status_code` 参数在 WebSocket 中不被支持，对于 WebSocket，始终会使用 403（禁止）状态码。

另外，您可能希望将未认证的用户重定向到其他页面。

```python
from starlette.authentication import requires


async def homepage(request):
    ...


@requires('authenticated', redirect='homepage')
async def dashboard(request):
    ...
```

在重定向用户时，您重定向的页面将包含用户最初请求的 URL，作为 `next` 查询参数：

```python
from starlette.authentication import requires
from starlette.responses import RedirectResponse


@requires('authenticated', redirect='login')
async def admin(request):
    ...


async def login(request):
    if request.method == "POST":
        # 现在用户已认证，
        # 我们可以将他们重定向到原始请求的目标
        if request.user.is_authenticated:
            next_url = request.query_params.get("next")
            if next_url:
                return RedirectResponse(next_url)
            return RedirectResponse("/")
```

对于基于类的端点，您应该将装饰器应用于类中的方法。

```python
from starlette.authentication import requires
from starlette.endpoints import HTTPEndpoint


class Dashboard(HTTPEndpoint):
    @requires("authenticated")
    async def get(self, request):
        ...
```

## 自定义身份验证错误响应

当身份验证后端引发 `AuthenticationError` 时，您可以自定义发送的错误响应：

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.authentication import AuthenticationMiddleware
from starlette.requests import Request
from starlette.responses import JSONResponse


def on_auth_error(request: Request, exc: Exception):
    return JSONResponse({"error": str(exc)}, status_code=401)

app = Starlette(
    middleware=[
        Middleware(AuthenticationMiddleware, backend=BasicAuthBackend(), on_error=on_auth_error),
    ],
)
```
