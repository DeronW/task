# 发布

Task 的发布流程是在 [GoReleaser][goreleaser] 的帮助下完成的。本地调用 Taskfile 的 `test-release` 任务可以测试发布流程。

[GitHub Actions](https://github.com/go-task/task/actions) 会在新 tag 推送到 master 分支的时候，自动发布产出物（原生的 DEB 和 RPM 包）。

从 v3.15.0 开始，本地切换到指定 tag 后，可以重新编译和校验原生的可执行文件，并调用 `goreleaser build`，
使用的 Go 版本是上面 Github Actions 中指定的版本。

# Homebrew

Goreleaser 会自动推送一个新的提交到 [Homebrew tap][homebrewtap] 仓库的 [Formula/go-task.rb][gotaskrb] 文件，以此完成新版本的发布。

# npm

发布 npm 新版本，需要更新 [`package.json`][packagejson] 文件中的 version，然后执行 `task npm:publish` 来进行发布。

# Snapcraft

[snap package][snappackage] 发布新版本需要手动执行下面步骤：

- 更新 [snapcraft.yaml][snapcraftyaml] 文件中的版本。
- 把新的 `amd64`, `armhf` 和 `arm64` 移动到 [Snapcraft dashboard][snapcraftdashboard] 的稳定通道。

# Scoop

Scoop 是一个 Windows 系统的命令行包管理工具。
Scoop 的包由社区维护。
Scoop 的维护人通常会在[这个文件](https://github.com/lukesampson/scoop-extras/blob/master/bucket/task.json)里维护版本。
如果发现 Task 版本是旧的，请提交一个 issue 通知我们。

# Nix

Nix 安装由社区维护。Nix 包的维护人员通常会在[这个文件](https://github.com/NixOS/nixpkgs/blob/nixos-unstable/pkgs/development/tools/go-task/default.nix)里维护版本。
如果发现 Task 版本是旧的，请提交一个 issue 通知我们。

[goreleaser]: https://goreleaser.com/
[homebrewtap]: https://github.com/go-task/homebrew-tap
[gotaskrb]: https://github.com/go-task/homebrew-tap/blob/master/Formula/go-task.rb
[packagejson]: https://github.com/go-task/task/blob/master/package.json#L3
[snappackage]: https://github.com/go-task/snap
[snapcraftyaml]: https://github.com/go-task/snap/blob/master/snap/snapcraft.yaml#L2
[snapcraftdashboard]: https://snapcraft.io/task/releases
