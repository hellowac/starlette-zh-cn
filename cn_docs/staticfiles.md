
Starlette 还包括一个 `StaticFiles` 类，用于提供给定目录中的文件服务：

### StaticFiles

签名：`StaticFiles(directory=None, packages=None, html=False, check_dir=True, follow_symlink=False)`

* `directory` - 一个字符串或 [os.PathLike][pathlike]，表示目录路径。
* `packages` - 一个字符串列表或包含字符串元组的列表，表示 Python 包。
* `html` - 以 HTML 模式运行。如果目录中存在 `index.html` 文件，则自动加载该文件。
* `check_dir` - 确保在实例化时目录存在。默认为 `True`。
* `follow_symlink` - 一个布尔值，指示是否应跟随文件和目录的符号链接。默认为 `False`。

您可以将此 ASGI 应用程序与 Starlette 的路由结合使用，以提供全面的静态文件服务。

```python
from starlette.applications import Starlette
from starlette.routing import Mount
from starlette.staticfiles import StaticFiles


routes = [
    ...
    Mount('/static', app=StaticFiles(directory='static'), name="static"),
]

app = Starlette(routes=routes)
```

对于不匹配的请求，静态文件将返回 "404 Not found" 或 "405 Method not allowed" 响应。如果在 HTML 模式下存在 `404.html` 文件，它将作为 404 响应显示。

`packages` 选项可用于包含 Python 包内的 "static" 目录。Python 的 "bootstrap4" 包就是一个例子。

```python
from starlette.applications import Starlette
from starlette.routing import Mount
from starlette.staticfiles import StaticFiles


routes=[
    ...
    Mount('/static', app=StaticFiles(directory='static', packages=['bootstrap4']), name="static"),
]

app = Starlette(routes=routes)
```

默认情况下，`StaticFiles` 会在每个包中查找 `statics` 目录，您可以通过指定字符串元组来更改默认目录。

```python
routes=[
    ...
    Mount('/static', app=StaticFiles(packages=[('bootstrap4', 'static')]), name="static"),
]
```

您可能更喜欢将静态文件直接包含在 "static" 目录中，而不是使用 Python 打包来包含静态文件，但它对于打包可重用组件可能是有用的。

[pathlike]: https://docs.python.org/3/library/os.html#os.PathLike
