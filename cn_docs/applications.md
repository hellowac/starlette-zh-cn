Starlette 包含一个应用类 `Starlette`，将其和其他功能优雅地结合在一起。

```python
from contextlib import asynccontextmanager

from starlette.applications import Starlette
from starlette.responses import PlainTextResponse
from starlette.routing import Route, Mount, WebSocketRoute
from starlette.staticfiles import StaticFiles


def homepage(request):
    return PlainTextResponse('Hello, world!')

def user_me(request):
    username = "John Doe"
    return PlainTextResponse('Hello, %s!' % username)

def user(request):
    username = request.path_params['username']
    return PlainTextResponse('Hello, %s!' % username)

async def websocket_endpoint(websocket):
    await websocket.accept()
    await websocket.send_text('Hello, websocket!')
    await websocket.close()

@asynccontextmanager
async def lifespan(app):
    print('Startup')
    yield
    print('Shutdown')


routes = [
    Route('/', homepage),
    Route('/user/me', user_me),
    Route('/user/{username}', user),
    WebSocketRoute('/ws', websocket_endpoint),
    Mount('/static', StaticFiles(directory="static")),
]

app = Starlette(debug=True, routes=routes, lifespan=lifespan)
```

### 实例化应用

**Instantiating the application**


::: starlette.applications.Starlette
    :docstring:

### 在应用实例上存储状态

您可以使用通用的 `app.state` 属性在应用实例上存储任意额外状态。

例如：

```python
app.state.ADMIN_EMAIL = 'admin@example.org'
```

### 访问应用实例

当有 `request` 可用时（例如在端点和中间件中），可以通过 `request.app` 访问应用实例。