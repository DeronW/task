# Task

<div align="center">
  <img id="logo" src="https://taskfile.dev/img/logo.svg" height="250px" width="250px" />
</div>

Task 是一个任务运行/构建工具，目的是为了方便使用，例如 [GNU Make][make]。

因为使用 [Go][go] 开发，Task 是一个独立的二进制文件，没有其它依赖，所以你不需要那些复杂的安装流程即可使用。

[安装](installation.md) 完成后，只需要创建一个 [YAML][yaml] 文件来描述你需要执行的任务，并把这个文件命名为 `Taskfile.yml`:

```yaml title="Taskfile.yml"
version: "3"

tasks:
  hello:
    cmds:
      - echo 'Hello World from Task!'
    silent: true
```

然后在终端中运行 `task hello` 来调用命令。

上面的例子只是个开始，通过查看 [usage](/usage) 来获取完整的模型文档和 Task 特性。

## 特性

- [易于安装](installation.md)：只需要下载二进制文件，添加到 `$PATH` 路径即可！也可以通过 [Homebrew][homebrew][Snapcraft][snapcraft], 或者 [Scoop][scoop] 安装。
- 可以在 CI 中使用，只要添加 [这个命令](installation.md#install-script) 到 CI 安装脚本中，然后就可以把 Task 当做 CI 的一个功能来使用了。
- 真正跨平台：大部分构建工具只能在 Linux 或 macOS 上稳定运行，Task 同时还支持 Windows 多谢 [Go 的 Shell 解释器](sh)。
- 对代码版本友好：很容易 [跳过任务执行](/usage#prevent-unnecessary-work) 如果待构建的的文件没有发生变化的话（可以基于文件的时间戳或文件内容）

[make]: https://www.gnu.org/software/make/
[go]: https://go.dev/
[yaml]: http://yaml.org/
[homebrew]: https://brew.sh/
[snapcraft]: https://snapcraft.io/
[scoop]: https://scoop.sh/
[sh]: https://github.com/mvdan/sh
