
Starlette 拥有一个快速增长的开发者社区，他们正在构建与 Starlette 集成的工具、依赖于 Starlette 的工具等。

以下是一些第三方插件：

## 插件

### Apitally

<a href="https://github.com/apitally/python-client" target="_blank">GitHub</a> | 
<a href="https://docs.apitally.io/frameworks/starlette" target="_blank">文档</a>

为 Starlette（以及其他框架）提供简单的流量、错误和响应时间监控，外加 API 密钥和权限管理。

### Authlib

<a href="https://github.com/lepture/Authlib" target="_blank">GitHub</a> | 
<a href="https://docs.authlib.org/en/latest/" target="_blank">文档</a>

一个构建 OAuth 和 OpenID Connect 客户端与服务器的终极 Python 库。查看如何与 [Starlette 集成](https://docs.authlib.org/en/latest/client/starlette.html)。

### ChannelBox

<a href="https://github.com/Sobolev5/channel-box" target="_blank">GitHub</a>

一个 websocket 广播解决方案。可以从代码中的任何部分向通道组发送消息。
查看 <a href="https://channel-box.andrey-sobolev.ru/" target="_blank">MySimpleChat</a>，这是一个使用 `channel-box` 和 `starlette` 构建的简单聊天应用。

### Imia

<a href="https://github.com/alex-oleshkevich/imia" target="_blank">GitHub</a>

为 Starlette 提供的认证框架，支持可插拔的认证器和登录/登出流程。

### Mangum

<a href="https://github.com/erm/mangum" target="_blank">GitHub</a>

用于 AWS Lambda 和 API Gateway 的无服务器 ASGI 适配器。

### Nejma

<a href="https://github.com/taoufik07/nejma" target="_blank">GitHub</a>

使用 websockets 管理和发送消息到通道组。
查看 <a href="https://github.com/taoufik07/nejma-chat" target="_blank">nejma-chat</a>，这是一个使用 `nejma` 和 `starlette` 构建的简单聊天应用。

### Scout APM

<a href="https://github.com/scoutapp/scout_apm_python" target="_blank">GitHub</a>

一个 APM（应用性能监控）解决方案，可以对您的应用进行检测以发现性能瓶颈。

### SpecTree

<a href="https://github.com/0b01001001/spectree" target="_blank">GitHub</a>

使用 Python 注解生成 OpenAPI 规范文档并验证请求和响应。减少样板代码（无需 YAML）。

### Starlette APISpec

<a href="https://github.com/Woile/starlette-apispec" target="_blank">GitHub</a>

为 Starlette 提供简单的 APISpec 集成。  
通过在端点的文档字符串中以 YAML 格式声明 OpenAPI (Swagger) 模式，为使用 Starlette 构建的 REST API 添加文档支持。

### Starlette Compress

<a href="https://github.com/Zaczero/starlette-compress" target="_blank">GitHub</a>

Starlette-Compress 是一个快速简单的中间件，用于压缩 Starlette 的响应。  
它支持 ZStd、Brotli 和 GZip 压缩算法，并提供合理的默认配置。

### Starlette Context

<a href="https://github.com/tomwojcik/starlette-context" target="_blank">GitHub</a>

用于 Starlette 的中间件，允许您存储和访问请求的上下文数据。  
可与日志记录结合使用，以便日志能够自动包含请求头中的信息，如 `x-request-id` 或 `x-correlation-id`。

### Starlette Cramjam

<a href="https://github.com/developmentseed/starlette-cramjam" target="_blank">GitHub</a>

为 Starlette 提供的中间件，支持 **brotli**、**gzip** 和 **deflate** 压缩算法，且对依赖要求最低。

### Starlette OAuth2 API

<a href="https://gitlab.com/jorgecarleitao/starlette-oauth2-api" target="_blank">GitLab</a>

一个 Starlette 中间件，通过 JWT 添加身份验证和授权功能。  
完全依赖于认证提供者为客户端颁发访问令牌或身份令牌。

### Starlette Prometheus

<a href="https://github.com/perdy/starlette-prometheus" target="_blank">GitHub</a>

一个插件，用于提供暴露 [Prometheus](https://prometheus.io/) 指标的端点，基于其[官方 Python 客户端](https://github.com/prometheus/client_python)。

### Starlette WTF

<a href="https://github.com/muicss/starlette-wtf" target="_blank">GitHub</a>

一个用于将 Starlette 和 WTForms 集成的简单工具，其设计理念参考了优秀的 Flask-WTF 库。

### Starlette-Login

<a href="https://github.com/jockerz/Starlette-Login" target="_blank">GitHub</a> | 
<a href="https://starlette-login.readthedocs.io/en/stable/" target="_blank">文档</a>

为 Starlette 提供用户会话管理功能。  
处理常见任务，例如用户登录、登出，以及长时间保持用户会话状态。


### Starsessions

<a href="https://github.com/alex-oleshkevich/starsessions" target="_blank">GitHub</a>

一个替代的会话支持实现，具有可自定义的存储后端。

### webargs-starlette

<a href="https://github.com/sloria/webargs-starlette" target="_blank">GitHub</a>

基于 [webargs](https://github.com/marshmallow-code/webargs) 构建的 Starlette 声明式请求解析与验证工具。  
支持使用类型注解解析查询字符串、JSON、表单、头信息和 Cookie。

### DecoRouter

<a href="https://github.com/MrPigss/DecoRouter" target="_blank">GitHub</a>

为 Starlette 提供 FastAPI 风格的路由功能。  
支持使用装饰器生成路由表。

### Starception

<a href="https://github.com/alex-oleshkevich/starception" target="_blank">GitHub</a>

为 Starlette 应用提供美观的异常页面。

### Starlette-Admin

<a href="https://github.com/jowilf/starlette-admin" target="_blank">GitHub</a> | 
<a href="https://jowilf.github.io/starlette-admin" target="_blank">文档</a>

一个简单且可扩展的管理界面框架。  
基于 [Tabler](https://tabler.io/) 和 [Datatables](https://datatables.net/) 构建，能够快速为您的模型生成完全可定制的管理界面。  
支持将数据导出为多种格式（*CSV*、*PDF*、*Excel* 等），通过复杂查询（包括 `AND` 和 `OR` 条件）筛选数据，以及上传文件等功能。

### Vellox

<a href="https://github.com/junah201/vellox" target="_blank">GitHub</a>

适用于 GCP Cloud Functions 的无服务器 ASGI 适配器。

## Starlette Bridge

<a href="https://github.com/tarsil/starlette-bridge" target="_blank">GitHub</a> | 
<a href="https://starlette-bridge.tarsild.io/" target="_blank">文档</a>

随着 `on_startup` 和 `on_shutdown` 的弃用，Starlette Bridge 确保您仍然可以使用旧的事件声明方式，同时在内部为您自动创建 `lifespan`。  
这样可以为现有的包提供向后兼容性，同时保持 Starlette 新 `lifespan` 事件的完整性。

## 框架

### FastAPI

<a href="https://github.com/tiangolo/fastapi" target="_blank">GitHub</a> | 
<a href="https://fastapi.tiangolo.com/" target="_blank">文档</a>

高性能、易于学习、快速开发、适合生产环境的 Web API 框架。  
受 **APIStar** 早期服务器系统的启发，支持路由参数的类型声明，基于 OpenAPI 3.0.0+ 规范（结合 JSON Schema），并由 **Pydantic** 提供数据处理支持。

---

### Flama

<a href="https://github.com/vortico/flama" target="_blank">GitHub</a> | 
<a href="https://flama.dev/" target="_blank">文档</a>

Flama 是一个面向**数据科学**的框架，旨在快速构建现代化和可靠的**机器学习（ML）API**。  
该框架的主要目标是使部署 ML API 变得极为简单。通过 Flama，数据科学家可以用一行代码将 ML 模型快速转化为异步、自动生成文档的 API，仅需几秒钟！

Flama 提供直观的 CLI 和易学的开发理念，能够快速构建**高性能**的 GraphQL、REST 和 ML API。它是开发异步、**生产级**服务的理想解决方案，并支持 ML 模型的**自动部署**。

---

### Greppo

<a href="https://github.com/greppo-io/greppo" target="_blank">GitHub</a> | 
<a href="https://docs.greppo.io/" target="_blank">文档</a>

一个用于构建地理空间仪表板和 Web 应用的 Python 框架。  
Greppo 是开源的 Python 框架，能够快速集成数据、算法、可视化和交互式 UI。它提供 API 更新后端变量、重新计算逻辑，并将更改反映到前端（数据变更钩子）。

---

### Responder

<a href="https://github.com/taoufik07/responder" target="_blank">GitHub</a> | 
<a href="https://python-responder.org/en/latest/" target="_blank">文档</a>

异步 Web 服务框架。功能包括 Flask 风格路由表达、YAML 支持、OpenAPI 架构生成、后台任务、GraphQL 等。

---

### Starlette-apps

<a href="https://github.com/yourlabs/starlette-apps" target="_blank">GitHub</a>

使用简单的应用系统构建属于自己的框架，例如 [Django-GDAPS](https://gdaps.readthedocs.io/en/latest/) 或 [CakePHP](https://cakephp.org/)。

---

### Dark Star

<a href="https://lllama.github.io/dark-star" target="_blank">文档</a> | 
<a href="https://github.com/lllama/dark-star" target="_blank">GitHub</a>

一个简单的框架，旨在最小化获取 HTML 到浏览器所需的代码。  
将文件路径转换为 Starlette 路由，并将视图代码与模板放在一起。支持 [htmx](https://htmx.org) 来增强前端。

---

### Xpresso

<a href="https://github.com/adriangb/xpresso" target="_blank">GitHub</a> | 
<a href="https://xpresso-api.dev/" target="_blank">文档</a>

基于 Starlette、Pydantic 和 [di](https://github.com/adriangb/di) 构建的灵活且可扩展的 Web 框架。

---

### Ellar

<a href="https://github.com/eadwinCode/ellar" target="_blank">GitHub</a> | 
<a href="https://eadwincode.github.io/ellar/" target="_blank">文档</a>

Ellar 是一个用于构建快速、高效、可扩展的 RESTAPI 和服务器端应用的 ASGI Web 框架。  
提供高抽象级别的服务器端开发，结合了面向对象编程（OOP）和函数式编程（FP）的元素，受 Nestjs 启发。  
基于 **Starlette**、**Pydantic** 和 **Injector** 三个核心库构建。

---

### Apiman

一个扩展工具，便于在 Starlette 项目中轻松集成 Swagger/OpenAPI 文档，并提供 [SwaggerUI](http://swagger.io/swagger-ui/) 和 [RedocUI](https://rebilly.github.io/ReDoc/)。

<a href="https://github.com/strongbugman/apiman" target="_blank">GitHub</a>

---

### Starlette-Babel

通过 Babel 集成，提供翻译、本地化和时区支持。

<a href="https://github.com/alex-oleshkevich/starlette_babel" target="_blank">GitHub</a>

---

### Starlette-StaticResources

<a href="https://github.com/DavidVentura/starlette-static-resources" target="_blank">GitHub</a>

支持为静态数据挂载 [package resources](https://docs.python.org/3/library/importlib.resources.html#module-importlib.resources)，类似于 [StaticFiles](https://www.starlette.io/staticfiles/)。

---

### Sentry

<a href="https://github.com/getsentry/sentry-python" target="_blank">GitHub</a> | 
<a href="https://docs.sentry.io/platforms/python/guides/starlette/" target="_blank">文档</a>

Sentry 是一个软件错误检测工具，可提供解决性能问题和错误的可操作洞察。  
它帮助用户诊断、修复并优化 Python 调试，并与 Starlette 无缝集成。Sentry 的功能包括错误跟踪、性能分析、上下文信息以及警报/通知。

---

### Shiny

<a href="https://github.com/posit-dev/py-shiny" target="_blank">GitHub</a> | 
<a href="https://shiny.posit.co/py/" target="_blank">文档</a>

基于 Starlette 和 asyncio，Shiny 通过反应式编程让开发者能够轻松创建 Python Web 应用。  
Shiny 消除了手动状态管理的繁琐工作，能够在运行时自动确定应用的最佳执行路径，同时最小化重新渲染。这意味着 Shiny 既适用于简单仪表板，也支持全功能的 Web 应用。 