# 贡献

欢迎为 Task 做出贡献，在提交 PR 前请新阅读本文档。

## 开始前

- **检查已有工作** - 是否已经存在 PR？是否已有 issue 在讨论你想做的特性/变更？
  请确保你的工作中确实考虑了相关的讨论内容。
- **向后兼容** - 你的变更是否破坏了已经存在的 Task 文件？
  向后兼容的变更会更容易被合并进去。
  有没有一种方法可以保持这种兼容性？如果没有，请优先考虑创建一个 issue，
  然后在花时间编写 PR 前先对 API 的变更进行讨论。

## 1. 安装环节

- **Go** - Task 使用 [Go] 编写。我们始终要支持最新的两个大版本，
  所以确保你使用的是最新版本。
- **Node.js** - [Node.js] 用来启动 Task 的文档服务，如果要本地运行那么也需要安装。
- **Yarn** - [Yarn] 是 Node.js 的包管理工具，Task 用它管理 Node 依赖。

## 2. 功能变更

- **代码风格** - 试着保持已存在的代码风格并确保代码用 `gofmt` 格式化过。
  我们在 CI 中使用 `golangci-lint` 检查代码的风格的一致性和最佳实践。
  本地 Taskfile 也有一个 `lint` 命令可以执行检查。
- **文档** - 确保添加/更新了相关文档。查看下面 [文档更新](#文档更新) 章节。
- **测试** - 确保添加/更新了相关测试，并且在提交 PR 前已通过所有测试。
  查看下面 [编写测试](#编写测试)章节。

### 运行新功能

运行新功能，可以执行 `go run ./cmd/task`。
通过 `testdata` 目录中的测试 Taskfile 构建开发任务，
可以 `go run ./cmd/task --dir ./testdata/<my_test_dir> <task_name>`。

### 文档更新

Task 用 [Docusaurus] 托管文档服务。
本地可以通过 `task docs:setup` 和 `task docs:start` 运行 (需要 `nodejs` & `yarn`)。
所有内容使用 Markdown 编写，并放到 `docs/docs` 目录。
所有文档需要符合每行最多 80 个字符的限制。

修改时，考虑是否有修改 [使用](./usage.md) 的必要。
这里包含 Task 的使用说明和例子。
如果添加了新功能，试着在合适的位置添加一个新章节。
如果是更新已有功能，确保文档和所有的例子都已更新。
确保 [风格指南](./styleguide.md) 中例子都是正确的。

如果添加了新字段，命令，参数，确保更新到 [API](./api_reference.md)。
新字段需要添加到 [JSON Schema](https://github.com/go-task/task/blob/master/docs/static/schema.json)。
API 中的字段描述需要与 shcmea 中的内容完全匹配。

### 编写测试

大部分 Task 的测试在项目根目录的 `task_test.go` 文件中。
新添加的测试一般也放到这里。大部分测试在 `testdata` 目录中都有一个子目录，
这是用来保存测试数据的。

修改时，考虑是否需要添加新的测试。这些测试是保证新增功能未来也能持续运行的。
现有测试如果有必要，也需要更新来适配新的 Task 行为。

## 3. 提交代码

编写有意义的提交信息，避免一个 PR 上有太多提交。
大部分 PR 应该只有一个提交（尽管大型 PR 有充足的缘由使用多个提交）。
Git squash 和 rebase 也应好好利用。

## 4. 提交 PR

- **描述变更** - 确保包含了关于变更的综合描述。
- **Issue/PR 链接** - 链接到之前相关的 issue 或 PR。请描述当前工作与之前的不同之处。
- **例子** - 添加有用的例子来演示新功能的作用。
- **PR 草案** - 如果变更还未完成，但想纳入相关讨论，可以打开一个 PR 草案，
  添加一个备注并开始讨论。多用备注，少用 PR 描述，这样可以最后更新描述，并保留所有讨论。

## FAQ

> 我想功能，从哪开始?

查看 [open issues] 列表。我们有 [good first issue] 标签未首次参与贡献的人保留问题。

欢迎所有形式的工薪，不管是拼写错误或非常小的功能。甚至可以通过投票、备注的方式，
帮助回答问题或帮助其它 [社区项目](./community.md).

> 我卡主了，怎么寻求帮助?

如果有问题，可以到 [Discord server] 的 `#help` 拼到上求助。

---

[go]: https://go.dev
[install version 1.18+]: https://go.dev/doc/install
[node.js]: https://nodejs.org/en/
[yarn]: https://yarnpkg.com/
[docusaurus]: https://docusaurus.io
[open issues]: https://github.com/go-task/task/issues
[good first issue]: https://github.com/go-task/task/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22
[discord server]: https://discord.gg/6TY36E39UK
