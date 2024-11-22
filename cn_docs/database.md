Starlette 并不严格依赖于任何特定的数据库实现。

您可以将它与异步 ORM（例如 [GINO](https://python-gino.org/)）一起使用，或者使用常规的非异步端点，并与 [SQLAlchemy](https://www.sqlalchemy.org/) 集成。

在本指南中，我们将展示如何与 [`databases` 包](https://github.com/encode/databases) 集成，`databases` 包提供了 SQLAlchemy 核心支持，能够与多种不同的数据库驱动程序兼容。

以下是一个完整的示例，包含表定义、配置 `database.Database` 实例，以及与数据库交互的几个端点。

**.env**

```ini
DATABASE_URL=sqlite:///test.db
```

**app.py**

```python
import contextlib

import databases
import sqlalchemy
from starlette.applications import Starlette
from starlette.config import Config
from starlette.responses import JSONResponse
from starlette.routing import Route


# 从环境变量或 '.env' 文件中加载配置。
config = Config('.env')
DATABASE_URL = config('DATABASE_URL')


# 数据库表定义。
metadata = sqlalchemy.MetaData()

notes = sqlalchemy.Table(
    "notes",
    metadata,
    sqlalchemy.Column("id", sqlalchemy.Integer, primary_key=True),
    sqlalchemy.Column("text", sqlalchemy.String),
    sqlalchemy.Column("completed", sqlalchemy.Boolean),
)

database = databases.Database(DATABASE_URL)

@contextlib.asynccontextmanager
async def lifespan(app):
    await database.connect()
    yield
    await database.disconnect()

# 主要应用代码。
async def list_notes(request):
    query = notes.select()
    results = await database.fetch_all(query)
    content = [
        {
            "text": result["text"],
            "completed": result["completed"]
        }
        for result in results
    ]
    return JSONResponse(content)

async def add_note(request):
    data = await request.json()
    query = notes.insert().values(
       text=data["text"],
       completed=data["completed"]
    )
    await database.execute(query)
    return JSONResponse({
        "text": data["text"],
        "completed": data["completed"]
    })

routes = [
    Route("/notes", endpoint=list_notes, methods=["GET"]),
    Route("/notes", endpoint=add_note, methods=["POST"]),
]

app = Starlette(
    routes=routes,
    lifespan=lifespan,
)
```

最后，您需要创建数据库表。推荐使用 Alembic，关于它的内容我们会在 [迁移](#migrations) 部分做简要介绍。

## 查询

查询可以通过 [SQLAlchemy Core 查询](https://docs.sqlalchemy.org/en/latest/core/tutorial.html) 来执行。

以下方法受支持：

* `rows = await database.fetch_all(query)`
* `row = await database.fetch_one(query)`
* `async for row in database.iterate(query)`
* `await database.execute(query)`
* `await database.execute_many(query)`

## 事务

数据库事务可以通过装饰器、上下文管理器或低级 API 使用。

使用装饰器在端点上：

```python
@database.transaction()
async def populate_note(request):
    # 此数据库插入操作在事务中执行。
    # 它将被 `RuntimeError` 回滚。
    query = notes.insert().values(text="你看不到我", completed=True)
    await database.execute(query)
    raise RuntimeError()
```

使用上下文管理器：

```python
async def populate_note(request):
    async with database.transaction():
        # 此数据库插入操作在事务中执行。
        # 它将被 `RuntimeError` 回滚。
        query = notes.insert().values(text="你看不到我", completed=True)
        await request.database.execute(query)
        raise RuntimeError()
```

使用低级 API：

```python
async def populate_note(request):
    transaction = await database.transaction()
    try:
        # 此数据库插入操作在事务中执行。
        # 它将被 `RuntimeError` 回滚。
        query = notes.insert().values(text="你看不到我", completed=True)
        await database.execute(query)
        raise RuntimeError()
    except:
        await transaction.rollback()
        raise
    else:
        await transaction.commit()
```

## 测试隔离

在对使用数据库的服务进行测试时，我们需要确保一些事项。我们的要求应当是：

* 使用一个独立的数据库进行测试。
* 每次运行测试时创建一个新的测试数据库。
* 确保每个测试用例之间数据库的状态是隔离的。

以下是如何组织应用程序和测试，以满足这些要求：

```python
from starlette.applications import Starlette
from starlette.config import Config
import databases

config = Config(".env")

TESTING = config('TESTING', cast=bool, default=False)
DATABASE_URL = config('DATABASE_URL', cast=databases.DatabaseURL)
TEST_DATABASE_URL = DATABASE_URL.replace(database='test_' + DATABASE_URL.database)

# 在测试期间使用 'force_rollback'，确保在每个测试用例之间不持久化数据库更改。
if TESTING:
    database = databases.Database(TEST_DATABASE_URL, force_rollback=True)
else:
    database = databases.Database(DATABASE_URL)
```

我们仍然需要在测试运行期间设置 `TESTING`，并设置测试数据库。假设我们使用 `py.test`，以下是我们的 `conftest.py` 可能的结构：

```python
import pytest
from starlette.config import environ
from starlette.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy_utils import database_exists, create_database, drop_database

# 这会设置 `os.environ`，但提供了一些额外的保护。
# 如果我们将其放在应用程序导入之后，它会引发错误，
# 提示 'TESTING' 已经从环境中读取。
environ['TESTING'] = 'True'

import app


@pytest.fixture(scope="session", autouse=True)
def create_test_database():
  """
  每个测试用例创建一个干净的数据库。
  为了安全，如果数据库已经存在，我们应当中止测试。

  我们在这里使用 `sqlalchemy_utils` 包，提供一些帮助函数
  来一致地创建和删除数据库。
  """
  url = str(app.TEST_DATABASE_URL)
  engine = create_engine(url)
  assert not database_exists(url), '测试数据库已经存在，正在中止测试。'
  create_database(url)             # 创建测试数据库。
  metadata.create_all(engine)      # 创建表格。
  yield                            # 运行测试。
  drop_database(url)               # 删除测试数据库。


@pytest.fixture()
def client():
    """
    当在测试用例中使用 'client' fixture 时，我们将确保每个测试用例之间
    完成数据库回滚：

    def test_homepage(client):
        url = app.url_path_for('homepage')
        response = client.get(url)
        assert response.status_code == 200
    """
    with TestClient(app) as client:
        yield client
```

## 数据库迁移

几乎可以肯定，您需要使用数据库迁移来管理数据库的增量更改。为此，我们强烈推荐使用 [Alembic](https://alembic.sqlalchemy.org/en/latest/)，它是由 SQLAlchemy 的作者编写的。

```shell
$ pip install alembic
$ alembic init migrations
```

现在，您需要设置 Alembic 以引用配置的 `DATABASE_URL` 并使用您的表格元数据。

在 `alembic.ini` 中，删除以下行：

```shell
sqlalchemy.url = driver://user:pass@localhost/dbname
```

在 `migrations/env.py` 中，您需要设置 `'sqlalchemy.url'` 配置项和 `target_metadata` 变量。它应该像这样：

```python
# Alembic 配置对象
config = context.config

# 配置 Alembic 使用我们的 DATABASE_URL 和表格定义...
import app
config.set_main_option('sqlalchemy.url', str(app.DATABASE_URL))
target_metadata = app.metadata

...
```

然后，使用我们上面的笔记示例，创建初始修订版本：

```shell
alembic revision -m "Create notes table"
```

并在新文件（在 `migrations/versions` 目录下）中填充必要的指令：

```python
def upgrade():
    op.create_table(
        'notes',
        sqlalchemy.Column("id", sqlalchemy.Integer, primary_key=True),
        sqlalchemy.Column("text", sqlalchemy.String),
        sqlalchemy.Column("completed", sqlalchemy.Boolean),
    )

def downgrade():
    op.drop_table('notes')
```

然后运行您的第一次迁移。我们的笔记应用现在可以运行了！

```shell
alembic upgrade head
```

**在测试期间运行迁移**

确保每次创建测试数据库时都运行数据库迁移是一个良好的做法。这有助于捕捉迁移脚本中的任何问题，并确保测试在与生产数据库一致的数据库状态下运行。

我们可以稍微调整 `create_test_database` fixture：

```python
from alembic import command
from alembic.config import Config
import app

...

@pytest.fixture(scope="session", autouse=True)
def create_test_database():
    url = str(app.DATABASE_URL)
    engine = create_engine(url)
    assert not database_exists(url), '测试数据库已经存在，正在中止测试。'
    create_database(url)             # 创建测试数据库。
    config = Config("alembic.ini")   # 运行迁移。
    command.upgrade(config, "head")
    yield                            # 运行测试。
    drop_database(url)               # 删除测试数据库。
```

[sqlalchemy-core]: https://docs.sqlalchemy.org/en/latest/core/
[alembic]: https://alembic.sqlalchemy.org/en/latest/
