Starlette 支持生成 API 模式，如广泛使用的 [OpenAPI 规范][openapi]（前身为 "Swagger"）。

模式生成通过检查应用程序中的路由（通过 `app.routes`）以及使用端点上的文档字符串或其他属性来确定完整的 API 模式。

Starlette 并不依赖于任何特定的模式生成或验证工具，但包含了一个简单的实现，基于文档字符串生成 OpenAPI 模式。

```python
from starlette.applications import Starlette
from starlette.routing import Route
from starlette.schemas import SchemaGenerator


schemas = SchemaGenerator(
    {"openapi": "3.0.0", "info": {"title": "Example API", "version": "1.0"}}
)

def list_users(request):
    """
    responses:
      200:
        description: A list of users.
        examples:
          [{"username": "tom"}, {"username": "lucy"}]
    """
    raise NotImplementedError()


def create_user(request):
    """
    responses:
      200:
        description: A user.
        examples:
          {"username": "tom"}
    """
    raise NotImplementedError()


def openapi_schema(request):
    return schemas.OpenAPIResponse(request=request)


routes = [
    Route("/users", endpoint=list_users, methods=["GET"]),
    Route("/users", endpoint=create_user, methods=["POST"]),
    Route("/schema", endpoint=openapi_schema, include_in_schema=False)
]

app = Starlette(routes=routes)
```

现在，我们可以在 "/schema" 端点访问 OpenAPI 模式。

您还可以直接通过 `.get_schema(routes)` 生成 API 模式：

```python
schema = schemas.get_schema(routes=app.routes)
assert schema == {
    "openapi": "3.0.0",
    "info": {"title": "Example API", "version": "1.0"},
    "paths": {
        "/users": {
            "get": {
                "responses": {
                    200: {
                        "description": "A list of users.",
                        "examples": [{"username": "tom"}, {"username": "lucy"}],
                    }
                }
            },
            "post": {
                "responses": {
                    200: {"description": "A user.", "examples": {"username": "tom"}}
                }
            },
        },
    },
}
```

您可能还希望能够打印出 API 模式，以便使用工具生成 API 文档。

```python
if __name__ == '__main__':
    assert sys.argv[-1] in ("run", "schema"), "Usage: example.py [run|schema]"

    if sys.argv[-1] == "run":
        uvicorn.run("example:app", host='0.0.0.0', port=8000)
    elif sys.argv[-1] == "schema":
        schema = schemas.get_schema(routes=app.routes)
        print(yaml.dump(schema, default_flow_style=False))
```

### 第三方包

#### [starlette-apispec][starlette-apispec]

Starlette 的简单 APISpec 集成，支持一些对象序列化库。

[openapi]: https://github.com/OAI/OpenAPI-Specification
[starlette-apispec]: https://github.com/Woile/starlette-apispec
