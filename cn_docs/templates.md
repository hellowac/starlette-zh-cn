Starlette 并没有与任何特定的模板引擎 _严格_ 绑定，但 Jinja2 是一个非常好的选择。

### Jinja2Templates

签名：`Jinja2Templates(directory, context_processors=None, **env_options)`

* `directory` - 一个字符串、[os.Pathlike][pathlike] 或一个表示目录路径的字符串列表或 [os.Pathlike][pathlike]。
* `context_processors` - 一个函数列表，这些函数返回一个字典，用于添加到模板上下文中。
* `**env_options` - 传递给 Jinja2 环境的附加关键字参数。

Starlette 提供了一种简单的方式来配置 `jinja2`。这通常是您默认希望使用的方式。

```python
from starlette.applications import Starlette
from starlette.routing import Route, Mount
from starlette.templating import Jinja2Templates
from starlette.staticfiles import StaticFiles


templates = Jinja2Templates(directory='templates')

async def homepage(request):
    return templates.TemplateResponse(request, 'index.html')

routes = [
    Route('/', endpoint=homepage),
    Mount('/static', StaticFiles(directory='static'), name='static')
]

app = Starlette(debug=True, routes=routes)
```

请注意，传入的 `request` 实例必须作为模板上下文的一部分包含在内。

Jinja2 模板上下文将自动包含一个 `url_for` 函数，因此我们可以正确地在应用程序内进行页面间的超链接。

例如，我们可以在 HTML 模板中链接静态文件：

```html
<link href="{{ url_for('static', path='/css/bootstrap.min.css') }}" rel="stylesheet" />
```

如果您想使用 [自定义过滤器][jinja2]，您需要更新 `Jinja2Templates` 的 `env` 属性：

```python
from commonmark import commonmark
from starlette.templating import Jinja2Templates

def marked_filter(text):
    return commonmark(text)

templates = Jinja2Templates(directory='templates')
templates.env.filters['marked'] = marked_filter
```

## 使用自定义 jinja2.Environment 实例

Starlette 还接受一个预配置的 [`jinja2.Environment`](https://jinja.palletsprojects.com/en/3.0.x/api/#api) 实例。

```python
import jinja2
from starlette.templating import Jinja2Templates

env = jinja2.Environment(...)
templates = Jinja2Templates(env=env)
```


## 上下文处理器

上下文处理器是一个函数，返回一个字典，该字典将合并到模板上下文中。每个函数只接受一个参数 `request`，并且必须返回一个字典，添加到上下文中。

模板处理器的一个常见用例是将共享变量扩展到模板上下文中。

```python
import typing
from starlette.requests import Request

def app_context(request: Request) -> typing.Dict[str, typing.Any]:
    return {'app': request.app}
```

### 注册上下文模板

将上下文处理器传递给 `Jinja2Templates` 类的 `context_processors` 参数。

```python
import typing

from starlette.requests import Request
from starlette.templating import Jinja2Templates

def app_context(request: Request) -> typing.Dict[str, typing.Any]:
    return {'app': request.app}

templates = Jinja2Templates(
    directory='templates', context_processors=[app_context]
)
```

!!! info

    不支持将异步函数作为上下文处理器。

## 测试模板响应

在使用测试客户端时，模板响应包括 `.template` 和 `.context` 属性。

```python
from starlette.testclient import TestClient


def test_homepage():
    client = TestClient(app)
    response = client.get("/")
    assert response.status_code == 200
    assert response.template.name == 'index.html'
    assert "request" in response.context
```

## 自定义 Jinja2 环境

`Jinja2Templates` 接受 Jinja2 `Environment` 支持的所有选项。这将允许您更好地控制 Starlette 创建的 `Environment` 实例。

有关 `Environment` 可用选项的列表，您可以查看 Jinja2 文档 [这里](https://jinja.palletsprojects.com/en/3.0.x/api/#jinja2.Environment)。

```python
from starlette.templating import Jinja2Templates


templates = Jinja2Templates(directory='templates', autoescape=False, auto_reload=True)
```

## 异步模板渲染

Jinja2 支持异步模板渲染，但通常我们建议您避免在模板中执行会调用数据库查询或其他 I/O 操作的逻辑。

相反，我们建议您确保端点执行所有 I/O 操作，例如，在视图中严格评估数据库查询，并将最终结果包含在上下文中。

[jinja2]: https://jinja.palletsprojects.com/en/3.0.x/api/?highlight=environment#writing-filters
[pathlike]: https://docs.python.org/3/library/os.html#os.PathLike
