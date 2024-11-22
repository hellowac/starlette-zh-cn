
Starlette 支持 HTTP/2 和 HTTP/3 服务器推送，使得可以将资源推送到客户端，从而加快页面加载时间。

### `Request.send_push_promise`

用于启动资源的服务器推送。如果服务器推送不可用，则此方法不执行任何操作。

签名：`send_push_promise(path)`

* `path` - 表示资源路径的字符串。

```python
from starlette.applications import Starlette
from starlette.responses import HTMLResponse
from starlette.routing import Route, Mount
from starlette.staticfiles import StaticFiles


async def homepage(request):
    """
    主页使用服务器推送来传送样式表。
    """
    await request.send_push_promise("/static/style.css")
    return HTMLResponse(
        '<html><head><link rel="stylesheet" href="/static/style.css"/></head></html>'
    )

routes = [
    Route("/", endpoint=homepage),
    Mount("/static", StaticFiles(directory="static"), name="static")
]

app = Starlette(routes=routes)
```
