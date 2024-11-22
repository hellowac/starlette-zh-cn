
Starlette 提供了一个 `Request` 类，为传入的请求提供了更友好的接口，而无需直接访问 ASGI 的 `scope` 和接收通道。

### Request 类

签名：`Request(scope, receive=None)`

```python
from starlette.requests import Request
from starlette.responses import Response


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    request = Request(scope, receive)
    content = '%s %s' % (request.method, request.url.path)
    response = Response(content, media_type='text/plain')
    await response(scope, receive, send)
```

`Request` 提供了映射接口，因此可以像使用 `scope` 一样使用它。

例如：`request['path']` 会返回 ASGI 的路径。

如果不需要访问请求体，可以在不提供 `receive` 参数的情况下实例化请求对象。

#### 方法

可以通过 `request.method` 访问请求方法。

#### URL

可以通过 `request.url` 访问请求的 URL。

该属性是一个类字符串对象，可以解析 URL 的所有组件。

例如：`request.url.path`、`request.url.port`、`request.url.scheme`。

#### Headers

请求头以不可变、大小写不敏感的多值字典形式呈现。

例如：`request.headers['content-type']`

#### 查询参数

查询参数以不可变多值字典形式呈现。

例如：`request.query_params['search']`

#### 路径参数

路由路径参数以字典形式呈现。

例如：`request.path_params['username']`

#### 客户端地址

客户端的远程地址以命名的二元组 `request.client`（或 `None`）形式呈现。

- 主机名或 IP 地址：`request.client.host`
- 客户端连接的端口号：`request.client.port`

#### Cookies

Cookies 以常规字典形式呈现。

例如：`request.cookies.get('mycookie')`

如果遇到无效的 Cookie，将忽略该值。（遵循 RFC2109）

#### 请求体（Body）

可以通过以下多种接口获取请求体内容：

- **以字节形式返回请求体**：`await request.body()`
- **将请求体解析为表单数据或多部分数据**：`async with request.form() as form:`
- **将请求体解析为 JSON 数据**：`await request.json()`

您还可以使用 `async for` 语法，以流的形式访问请求体：

```python
from starlette.requests import Request
from starlette.responses import Response


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    request = Request(scope, receive)
    body = b''
    async for chunk in request.stream():
        body += chunk
    response = Response(body, media_type='text/plain')
    await response(scope, receive, send)
```

如果使用 `.stream()`，字节块会直接提供而不会将整个请求体存储到内存中。在这种情况下，后续调用 `.body()`、`.form()` 或 `.json()` 都会引发错误。

某些情况下，例如长轮询或流式响应，您可能需要判断客户端是否已断开连接。可以通过以下方法确定连接状态：

```python
disconnected = await request.is_disconnected()
```

#### 请求文件（Request Files）

请求文件通常以多部分表单数据的形式发送（`multipart/form-data`）。

签名：`request.form(max_files=1000, max_fields=1000)`

可以通过 `max_files` 和 `max_fields` 参数配置最大字段数或最大文件数：

```python
async with request.form(max_files=1000, max_fields=1000):
    ...
```

!!! info

    出于安全考虑，这些限制防止解析过多的空字段或文件导致 CPU 和内存资源耗尽，从而引发拒绝服务攻击（DoS）。允许无限制的字段或文件是不安全的。

调用 `async with request.form() as form` 时，返回的是一个不可变多值字典 `starlette.datastructures.FormData`，其中包含文件上传和文本输入。文件上传项表示为 `starlette.datastructures.UploadFile` 实例。

`UploadFile` 具有以下属性：

- **`filename`**：一个字符串，表示上传的原始文件名，例如 `myimage.jpg`；如果不可用，则为 `None`。
- **`content_type`**：一个字符串，表示内容类型（MIME 类型/媒体类型），例如 `image/jpeg`；如果不可用，则为 `None`。
- **`file`**：一个 <a href="https://docs.python.org/3/library/tempfile.html#tempfile.SpooledTemporaryFile" target="_blank">`SpooledTemporaryFile`</a>（类似文件的对象）。您可以直接将其传递给需要“类似文件”对象的其他函数或库。
- **`headers`**：一个 `Headers` 对象。通常仅包含 `Content-Type` 头，但如果多部分字段中包含其他头，则也会包含在此处。注意，这些头与 `Request.headers` 中的头无关。
- **`size`**：一个整数，表示上传文件的大小（字节）。该值从请求内容中计算得出，比 `Content-Length` 头更可靠；如果未设置，则为 `None`。

`UploadFile` 提供以下 `async` 方法，这些方法底层调用内部 `SpooledTemporaryFile` 的对应方法：

- **`async write(data)`**：将字节数据 `data` 写入文件。
- **`async read(size)`**：读取文件的 `size` 字节。
- **`async seek(offset)`**：将文件指针移动到 `offset` 字节位置。例如，`await myfile.seek(0)` 会将指针移动到文件开头。
- **`async close()`**：关闭文件。

由于这些方法是异步方法，调用时需要使用 `await`。

例如，可以通过以下方式获取文件名和内容：

```python
async with request.form() as form:
    filename = form["upload_file"].filename
    contents = await form["upload_file"].read()
```

!!! 提示
    根据 [RFC-7578: 4.2](https://www.ietf.org/rfc/rfc7578.txt)，包含文件的表单数据部分应该在 `Content-Disposition` 头中包含 `name` 和 `filename` 字段，例如：  
    `Content-Disposition: form-data; name="user"; filename="somefile"`。  
    虽然根据 RFC-7578，`filename` 字段是可选的，但它帮助 Starlette 区分哪些数据应被视为文件。如果提供了 `filename` 字段，将创建一个 `UploadFile` 对象来访问底层文件；否则，表单数据部分将被解析为原始字符串。

#### 应用程序（Application）

可以通过 `request.app` 访问请求源自的 Starlette 应用。

#### 其他状态（Other State）

如果需要在请求中存储额外信息，可以使用 `request.state`。

例如：

```python
request.state.time_started = time.time()
```
