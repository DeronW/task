# Task

<div align="center">
  <img id="logo" src="https://taskfile.dev/img/logo.svg" height="250px" width="250px" />
</div>

Task 是一个运行/构建 task 的工具，一款更好用的工具，相对于比如说 [GNU Make][make]。

因为是 [Go][go] 开发的，所以 Task 只是一个二进制文件，没有其它依赖，所以不需要乱七八糟的安装流程，就为了一个构建工具。

一旦 [安装](installation.zh.md) 完成，只需要创建一个名为 `Taskfile.yml` 的 [YAML][yaml] 文件，描述你需要执行的 task:

```yaml title="Taskfile.yml"
version: "3"

tasks:
  hello:
    cmds:
      - echo 'Hello World from Task!'
    silent: true
```

然后在终端执行 `task hello` 来调用命令。

上面的例子只是个开始，通过查看 [使用](/usage) 来了解完整的配置文档和 Task 特性。

## 特性

- [易于安装](installation.zh.md)：只需要下载二进制文件，添加到 `$PATH` 路径即可！也可以通过 [Homebrew][homebrew][Snapcraft][snapcraft], 或者 [Scoop][scoop] 安装。
- 可以在 CI 中使用：只要添加 [这个命令](installation.zh.md#安装脚本) 到 CI 安装脚本中，然后就可以把 Task 当做 CI 的一个功能来使用了。
- 真正跨平台：大部分构建工具只适配了 Linux 或 macOS，而 Task 还支持 Windows，多谢 [这个 Go 解释器](sh)。
- 代码版本友好：方便的 [跳过任务执行](/usage.zh.md#节省非必要工作) 如果与上次构建相比源文件没有变化的话（基于文件的时间戳或文件内容）

[make]: https://www.gnu.org/software/make/
[go]: https://go.dev/
[yaml]: http://yaml.org/
[homebrew]: https://brew.sh/
[snapcraft]: https://snapcraft.io/
[scoop]: https://scoop.sh/
[sh]: https://github.com/mvdan/sh
