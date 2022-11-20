---
slug: /community/
sidebar_position: 6
---

# 社区

社区已经完成了一些提升 Task 生态的工作，
改善 Task 生态的的一些工作已经由社区完成，均可以通过安装方法或与编辑器集成来使用。我（指作者）非常感谢所有帮助提升整体体验的开发者。

## 编辑器集成

### JSON Schema

[@KROSF](https://github.com/KROSF) 编写了 JSON Schema [在这查看 Gist](https://gist.github.com/KROSF/c5435acf590acd632f71bb720f685895),
后来通过 [@Crandel](https://github.com/Crandel) 成为官方推荐 [https://json.schemastore.org/taskfile.json](https://json.schemastore.org/taskfile.json).
如果想要增加功能可以修改 [这个文件](https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/taskfile.json) 然后提交 pull requests。
有些编辑器，比如 Visual Studio Code 会自动更新 Schema。

### Visual Studio Code 扩展

另外，在开发 Visual Studio Code 扩展过程中，还有一些工作由 [@paulvarache](https://github.com/paulvarache) 完成，
代码在 [这里](https://github.com/paulvarache/vscode-taskfile)
并发布到了 [这里](https://marketplace.visualstudio.com/items?itemName=paulvarache.vscode-taskfile)。

### Sublime Text 4 包

通过 Sublime Text 的命令面板有一个简便的安装运行方法。这个包是由 [@biozz](https://github.com/biozz) 开发的，
源代码在 [这里](https://github.com/biozz/sublime-taskfile)
并且发布到了包管理 [这里](https://packagecontrol.io/packages/Taskfile)。

### IntelliJ 插件

JetBrains IntelliJ 插件是由 [@lechuckroh](https://github.com/lechuckroh) 完成的，
代码在 [这里](https://github.com/lechuckroh/task-intellij-plugin)
并且发布到了 [这里](https://plugins.jetbrains.com/plugin/17058-taskfile)。

## 安装方法

有些安装方式是通过第三方维护的：

- [GitHub Actions](https://github.com/arduino/setup-task)
  by [@arduino](https://github.com/arduino)
- [AUR](https://aur.archlinux.org/packages/go-task-bin)
  by [@carlsmedstad](https://github.com/carlsmedstad)
- [Scoop](https://github.com/lukesampson/scoop-extras/blob/master/bucket/task.json)
- [Fedora](https://packages.fedoraproject.org/pkgs/golang-github-task/go-task/)
- [NixOS](https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/tools/go-task/default.nix)

## 更多

同时，谢谢所有 [代码贡献者](https://github.com/go-task/task/graphs/contributors)，
[资金贡献者](https://opencollective.com/task)，以及
[提交问题](https://github.com/go-task/task/issues?q=is%3Aissue) 和
[问题解答](https://github.com/go-task/task/discussions)。

如果你发现文档有哪些遗漏信息，欢迎提交 pull request。
