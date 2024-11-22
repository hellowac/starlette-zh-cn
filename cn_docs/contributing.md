# 贡献指南

感谢您对参与 **Starlette** 开发的兴趣！  
以下是您可以为该项目贡献的多种方式：

- 尝试使用 Starlette，并[报告您发现的错误/问题](https://github.com/encode/starlette/issues/new)
- [实现新功能](https://github.com/encode/starlette/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)
- [审查他人的拉取请求 (Pull Requests)](https://github.com/encode/starlette/pulls)
- 编写文档
- 参与讨论

---

## 报告错误或其他问题

发现 Starlette 应该支持的功能？  
遇到了一些意外的行为？  

贡献通常应从[讨论](https://github.com/encode/starlette/discussions)开始。  
可能的错误可以作为“潜在问题”讨论提出，功能请求可以作为“创意”讨论提出。之后，我们将判断是否需要将讨论升级为“问题”，或者是否考虑直接提交拉取请求。

请尽量详细描述问题。如果是错误报告，请提供尽可能多的信息，例如：

- 操作系统平台
- Python 版本
- 安装的依赖包及版本（使用 `python -m pip freeze` 命令）
- 相关代码片段
- 错误回溯信息

请务必尽量简化示例代码，仅保留最小可复现问题的部分。

---

## 开发

要开始开发 Starlette，请先在 GitHub 上**创建一个分叉 (fork)**，源仓库地址为：[Starlette 仓库](https://github.com/encode/starlette)。  

然后使用以下命令克隆您的分叉版本（将 `YOUR-USERNAME` 替换为您的 GitHub 用户名）：

```shell
$ git clone https://github.com/YOUR-USERNAME/starlette
```

克隆完成后，使用以下命令安装项目及其依赖：

```shell
$ cd starlette
$ scripts/install
```

---

## 测试与代码检查

我们使用自定义的 shell 脚本来自动化测试、代码检查和文档生成流程。

运行测试用例的命令为：

```shell
$ scripts/test
```

任何额外的参数都会传递给 `pytest`。有关更多信息，请参考 [pytest 文档](https://docs.pytest.org/en/latest/how-to/usage.html)。

例如，运行单个测试脚本：

```shell
$ scripts/test tests/test_application.py
```

运行代码自动格式化：

```shell
$ scripts/lint
```

如果想单独运行代码检查（它们也会作为 `scripts/test` 的一部分执行），可以使用以下命令：

```shell
$ scripts/check
```

## 文档编写

文档页面位于 `docs/` 文件夹下。

如果需要本地运行文档网站（便于预览修改内容），可以使用以下命令：

```shell
$ scripts/docs
```

---

## 解决构建/CI 失败

提交拉取请求后，测试套件会自动运行，其结果将在 GitHub 上显示。  
如果测试套件失败，点击“Details”链接，尝试定位失败原因。

<p align="center" style="margin: 0 0 10px">
  <img src="https://raw.githubusercontent.com/encode/starlette/master/docs/img/gh-actions-fail.png" alt='失败的 PR 提交状态'>
</p>

以下是一些常见的测试失败情况：

### 检查任务失败

<p align="center" style="margin: 0 0 10px">
  <img src="https://raw.githubusercontent.com/encode/starlette/master/docs/img/gh-actions-fail-check.png" alt='失败的 GitHub Actions 检查任务'>
</p>

如果此任务失败，表示存在代码格式问题或类型注解问题。  
可以通过任务输出日志查看失败原因，或者在终端运行以下命令检查问题：

```shell
$ scripts/check
```

建议运行 `$ scripts/lint` 尝试自动格式化代码。如果格式化成功，则提交相关更改。

### 文档任务失败

如果此任务失败，表示文档未能成功生成。这可能是因为无效的 Markdown 或 `mkdocs.yml` 配置缺失等原因。

### Python 3.X 任务失败

<p align="center" style="margin: 0 0 10px">
  <img src="https://raw.githubusercontent.com/encode/starlette/master/docs/img/gh-actions-fail-test.png" alt='失败的 GitHub Actions 测试任务'>
</p>

如果此任务失败，可能是单元测试未通过，或者未覆盖所有代码路径。

- 如果测试失败，覆盖率报告中会显示以下信息：

    `=== 1 failed, 435 passed, 1 skipped, 1 xfailed in 11.09s ===`

- 如果测试通过但覆盖率未达到要求的阈值，报告中会显示：

    `FAIL Required test coverage of 100% not reached. Total coverage: 99.00%`

---

## 发布版本

*此部分仅适用于 Starlette 维护者。*

发布新版本前，请创建一个包含以下内容的拉取请求：

1. **更新变更日志**：
    - 遵循 [keepachangelog](https://keepachangelog.com/en/1.0.0/) 格式。
    - 使用 [比较工具](https://github.com/encode/starlette/compare/) 将 `master` 与最新发布的标签进行对比，列出用户感兴趣的所有更改：
        - **必须**包含：新增、更改、弃用或移除的功能，以及错误修复。
        - **不应**包含：文档、测试或工具相关的更改。
        - 按影响/重要性降序排序条目。
        - 内容简洁明了，直击重点 🎯。
2. **版本号更新**：修改 `__version__.py` 文件。

    示例见 [#1600](https://github.com/encode/starlette/pull/1600)。

拉取请求合并后，创建一个  
[新版本发布](https://github.com/encode/starlette/releases/new)，包括：

- 标签版本：如 `0.13.3`
- 发布标题：`Version 0.13.3`
- 从变更日志复制的描述内容。

创建后，该版本会自动上传到 PyPI。

如果 PyPI 发布任务失败，可以通过以下命令手动发布：

```shell
$ scripts/publish
```
