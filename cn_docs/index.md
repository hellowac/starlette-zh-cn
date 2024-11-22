<p align="center">
  <a href="https://www.starlette.io/"><img width="420px" src="https://raw.githubusercontent.com/encode/starlette/master/docs/img/starlette.svg" alt='starlette'></a>
</p>
<p align="center">
    <em>✨ 闪亮的小 ASGI 框架. ✨</em>
</p>

---

[![Build Status](https://github.com/encode/starlette/workflows/Test%20Suite/badge.svg)](https://github.com/encode/starlette/actions)
[![Package version](https://badge.fury.io/py/starlette.svg)](https://pypi.python.org/pypi/starlette)
[![Supported Python Version](https://img.shields.io/pypi/pyversions/starlette.svg?color=%2334D058)](https://pypi.org/project/starlette)

---

**文档**: <a href="https://www.starlette.io/" target="_blank">英文</a> - <a href="https://hellowac.github.io/starlette-zh-cn/" target="_blank">中文</a>

**源码**: <a href="https://github.com/encode/starlette" target="_blank">https://github.com/encode/starlette</a>

---

# Starlette

Starlette 是一个轻量级的 [ASGI][asgi] 框架/工具包，非常适合用于构建 Python 异步 Web 服务。

它已经达到生产级别，并提供以下功能：

- 一个轻量、低复杂度的 HTTP Web 框架。
- WebSocket 支持。
- 进程内后台任务。
- 启动和关闭事件支持。
- 基于 `httpx` 的测试客户端。
- 提供 CORS、GZip、静态文件、流式响应支持。
- 会话和 Cookie 支持。
- 100% 测试覆盖率。
- 100% 类型注解代码库。
- 很少的硬性依赖。
- 兼容 `asyncio` 和 `trio` 后端。
- 在 [独立基准测试][techempower] 中表现优秀。

## 安装

```shell
$ pip install starlette
```

您还需要安装一个 ASGI 服务器，例如 [uvicorn](https://www.uvicorn.org/)、[daphne](https://github.com/django/daphne/) 或 [hypercorn](https://hypercorn.readthedocs.io/en/latest/)。

```shell
$ pip install uvicorn
```

## 示例

```python title="example.py"
from starlette.applications import Starlette
from starlette.responses import JSONResponse
from starlette.routing import Route


async def homepage(request):
    return JSONResponse({'hello': 'world'})

routes = [
    Route("/", endpoint=homepage)
]

app = Starlette(debug=True, routes=routes)
```

然后使用 Uvicorn 运行该应用：

```shell
$ uvicorn example:app
```

如需更完整的示例，请参考 [encode/starlette-example](https://github.com/encode/starlette-example)。

## 依赖

Starlette 仅需要 `anyio` 作为必需依赖，以下是可选依赖：

- [`httpx`][httpx] - 如果您需要使用 `TestClient`。
- [`jinja2`][jinja2] - 如果您需要使用 `Jinja2Templates`。
- [`python-multipart`][python-multipart] - 如果您需要支持表单解析（通过 `request.form()`）。
- [`itsdangerous`][itsdangerous] - 如果您需要支持 `SessionMiddleware`。
- [`pyyaml`][pyyaml] - 如果您需要支持 `SchemaGenerator`。

您可以通过以下命令一次性安装所有这些依赖项：

```shell
$ pip install starlette[full]
```

## 框架还是工具包

Starlette 被设计为既可以作为完整的框架使用，也可以作为 ASGI 工具包使用。它的任何组件都可以独立使用。

```python
from starlette.responses import PlainTextResponse


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = PlainTextResponse('Hello, world!')
    await response(scope, receive, send)
```

在 `example.py` 文件中运行 `app` 应用：

```shell
$ uvicorn example:app
INFO: Started server process [11509]
INFO: Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

使用 `--reload` 选项运行 uvicorn，可以在代码更改时自动重载。

## 模块化

Starlette 的模块化设计鼓励构建可复用的组件，这些组件可以在任何 ASGI 框架之间共享。这种设计应当能够催生一个共享中间件和可挂载应用的生态系统。

清晰的 API 分离还意味着更容易单独理解每个组件的功能。

---

<p align="center"><i>Starlette is <a href="https://github.com/encode/starlette/blob/master/LICENSE.md">BSD licensed</a> code.<br/>Designed & crafted with care.</i></br>&mdash; ⭐️ &mdash;</p>

[asgi]: https://asgi.readthedocs.io/en/latest/
[httpx]: https://www.python-httpx.org/
[jinja2]: https://jinja.palletsprojects.com/
[python-multipart]: https://andrew-d.github.io/python-multipart/
[itsdangerous]: https://itsdangerous.palletsprojects.com/
[sqlalchemy]: https://www.sqlalchemy.org
[pyyaml]: https://pyyaml.org/wiki/PyYAMLDocumentation
[techempower]: https://www.techempower.com/benchmarks/#hw=ph&test=fortune&l=zijzen-sf
