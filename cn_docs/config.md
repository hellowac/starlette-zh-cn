Starlette 鼓励将配置与代码严格分离，遵循 [十二因素应用模式](https://12factor.net/).

配置应存储在环境变量中，或者在一个不提交到源代码管理的 `.env` 文件中。

```python title="main.py"
from sqlalchemy import create_engine
from starlette.applications import Starlette
from starlette.config import Config
from starlette.datastructures import CommaSeparatedStrings, Secret

# 配置将从环境变量和/或 ".env" 文件中读取。
config = Config(".env")

DEBUG = config('DEBUG', cast=bool, default=False)
DATABASE_URL = config('DATABASE_URL')
SECRET_KEY = config('SECRET_KEY', cast=Secret)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=CommaSeparatedStrings)

app = Starlette(debug=DEBUG)
engine = create_engine(DATABASE_URL)
...
```

```shell title=".env"
# 请不要将此文件提交到源代码管理中。
# 例如，将 ".env" 包含在 `.gitignore` 文件中。
DEBUG=True
DATABASE_URL=postgresql://user:password@localhost:5432/database
SECRET_KEY=43n080musdfjt54t-09sdgr
ALLOWED_HOSTS=127.0.0.1, localhost
```

## 配置优先级

配置值读取的顺序如下：

* 从环境变量中读取。
* 从 `.env` 文件中读取。
* 从 `config` 中给出的默认值读取。

如果这些都没有匹配到，`config(...)` 会引发错误。

## 秘密

对于敏感密钥，`Secret` 类非常有用，因为它有助于减少值在回溯或其他代码 introspection 时泄漏的机会。

要获取 `Secret` 实例的值，您必须显式将其转换为字符串。应该仅在使用该值的地方进行转换。

```python
>>> from myproject import settings
>>> settings.SECRET_KEY
Secret('**********')
>>> str(settings.SECRET_KEY)
'98n349$%8b8-7yjn0n8y93T$23r'
```

!!! 提示

    您可以使用 `databases` 包中的 `DatabaseURL` 
    [这里](https://github.com/encode/databases/blob/ab5eb718a78a27afe18775754e9c0fa2ad9cd211/databases/core.py#L420)
    来存储数据库 URL，并避免它们在日志中泄漏。

## CommaSeparatedStrings

对于在单个配置键中存储多个值，`CommaSeparatedStrings` 类型非常有用。

```python
>>> from myproject import settings
>>> print(settings.ALLOWED_HOSTS)
CommaSeparatedStrings(['127.0.0.1', 'localhost'])
>>> print(list(settings.ALLOWED_HOSTS))
['127.0.0.1', 'localhost']
>>> print(len(settings.ALLOWED_HOSTS))
2
>>> print(settings.ALLOWED_HOSTS[0])
'127.0.0.1'
```

## 读取或修改环境变量

在某些情况下，您可能需要以编程方式读取或修改环境变量。这在测试中尤其有用，您可能希望覆盖环境中的某些键。

与其直接从 `os.environ` 读取或写入，您应使用 Starlette 的 `environ` 实例。这个实例是对标准 `os.environ` 的映射，它通过在读取配置后如果有任何环境变量被设置时抛出错误，额外为您提供保护。

如果您正在使用 `pytest`，则可以在 `tests/conftest.py` 中设置任何初始环境。

```python title="tests/conftest.py"
from starlette.config import environ

environ['DEBUG'] = 'TRUE'
```

## 读取带前缀的环境变量

您可以通过设置 `env_prefix` 参数来为环境变量指定命名空间。

```python title="myproject/settings.py"
import os

from starlette.config import Config

os.environ['APP_DEBUG'] = 'yes'
os.environ['ENVIRONMENT'] = 'dev'

config = Config(env_prefix='APP_')

DEBUG = config('DEBUG')  # 查找 APP_DEBUG，返回 "yes"
ENVIRONMENT = config('ENVIRONMENT')  # 查找 APP_ENVIRONMENT，抛出 KeyError，因为变量未定义
```

## 完整示例

构建大型应用程序可能很复杂。您需要适当分离配置和代码、在测试期间进行数据库隔离、分开测试和生产数据库等...

在这里，我们将展示一个完整的示例，演示如何开始构建应用程序的结构。

首先，让我们将设置、数据库表定义和应用程序逻辑分开：

```python title="myproject/settings.py"
from starlette.config import Config
from starlette.datastructures import Secret

config = Config(".env")

DEBUG = config('DEBUG', cast=bool, default=False)
SECRET_KEY = config('SECRET_KEY', cast=Secret)

DATABASE_URL = config('DATABASE_URL')
```

```python title="myproject/tables.py"
import sqlalchemy

# 数据库表定义
metadata = sqlalchemy.MetaData()

organisations = sqlalchemy.Table(
    ...
)
```

```python title="myproject/app.py"
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.sessions import SessionMiddleware
from starlette.routing import Route

from myproject import settings


async def homepage(request):
    ...

routes = [
    Route("/", endpoint=homepage)
]

middleware = [
    Middleware(
        SessionMiddleware,
        secret_key=settings.SECRET_KEY,
    )
]

app = Starlette(debug=settings.DEBUG, routes=routes, middleware=middleware)
```

接下来处理我们的测试配置。
我们希望在每次运行测试套件时创建一个新的测试数据库，并在测试完成后删除它。我们还希望确保：

```python title="tests/conftest.py"
from starlette.config import environ
from starlette.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy_utils import create_database, database_exists, drop_database

# 如果在导入 'settings' 后使用这行代码，它将抛出错误。
environ['DEBUG'] = 'TRUE'

from myproject import settings
from myproject.app import app
from myproject.tables import metadata


@pytest.fixture(autouse=True, scope="session")
def setup_test_database():
    """
    每次运行测试时创建一个干净的测试数据库。
    """
    url = settings.DATABASE_URL
    engine = create_engine(url)
    assert not database_exists(url), '测试数据库已经存在，终止测试。'
    create_database(url)             # 创建测试数据库。
    metadata.create_all(engine)      # 创建表。
    yield                            # 运行测试。
    drop_database(url)               # 删除测试数据库。


@pytest.fixture()
def client():
    """
    为测试用例提供一个 'client' 固定装置。
    """
    # 我们的固定装置在上下文管理器内创建。这确保了
    # 应用程序的生命周期在每个测试用例中都会运行。
    with TestClient(app) as test_client:
        yield test_client
```

[twelve-factor]: https://12factor.net/config
