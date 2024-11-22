
Starlette 应用程序可以注册一个生命周期处理器，用于处理在应用程序启动之前或应用程序关闭时需要运行的代码。

```python
import contextlib

from starlette.applications import Starlette


@contextlib.asynccontextmanager
async def lifespan(app):
    async with some_async_resource():
        print("在启动时运行！")
        yield
        print("在关闭时运行！")


routes = [
    ...
]

app = Starlette(routes=routes, lifespan=lifespan)
```

在生命周期处理器运行之前，Starlette 不会开始处理任何传入的请求。

生命周期的拆卸过程将在所有连接关闭并且任何正在处理的后台任务完成后运行。

考虑使用 [`anyio.create_task_group()`](https://anyio.readthedocs.io/en/stable/tasks.html) 来管理异步任务。

## 生命周期状态

生命周期有一个 `state` 的概念，它是一个字典，可以用来在生命周期和请求之间共享对象。

```python
import contextlib
from typing import AsyncIterator, TypedDict

import httpx
from starlette.applications import Starlette
from starlette.requests import Request
from starlette.responses import PlainTextResponse
from starlette.routing import Route


class State(TypedDict):
    http_client: httpx.AsyncClient


@contextlib.asynccontextmanager
async def lifespan(app: Starlette) -> AsyncIterator[State]:
    async with httpx.AsyncClient() as client:
        yield {"http_client": client}


async def homepage(request: Request) -> PlainTextResponse:
    client = request.state.http_client
    response = await client.get("https://www.example.com")
    return PlainTextResponse(response.text)


app = Starlette(
    lifespan=lifespan,
    routes=[Route("/", homepage)]
)
```

请求中接收到的 `state` 是生命周期处理器接收到的 `state` 的**浅拷贝**。

## 在测试中运行生命周期

你应该使用 `TestClient` 作为上下文管理器，以确保生命周期处理器被调用。

```python
from example import app
from starlette.testclient import TestClient


def test_homepage():
    with TestClient(app) as client:
        # 应用程序的生命周期在进入块时被调用。
        response = client.get("/")
        assert response.status_code == 200

    # 当退出块时，生命周期的拆卸过程会被执行。
```
