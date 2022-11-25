# 安装

Task 提供以下多种安装方式。

## 包管理工具

### Homebrew

如果在 macOS 或 Linux 已经安装了 [Homebrew][homebrew]，再安装 Task 只要这样：

```bash
brew install go-task/tap/go-task
```

### Snap

Task 可以通过 [Snapcraft][snapcraft] 安装，
但要注意你的 Linux 发行版需要符合 Snaps 的 classic 约束才能正确安装：

```bash
sudo snap install task --classic
```

### Chocolatey

如果 Windows 上安装了 [Chocolatey][choco]，再安装 Task 只要这样：

```bash
choco install go-task
```

这种方式由社区维护。

### Scoop

如果 Windows 上安装了 [Scoop][scoop]，再安装 Task 只要这样：

```cmd
scoop install task
```

这种方式由社区维护。新版 Task 发布后，需要过一段时间才能通过 Scoop 安装。

### AUR

如果是 Arch Linux 你可以用 [AUR](https://aur.archlinux.org/packages/go-task-bin) 安装，
只要配合你喜欢的包管理器，比如 `yay`, `pacaur` 或 `yaourt`：

```cmd
yay -S go-task-bin
```

也可以直接下载 [包文件](https://aur.archlinux.org/packages/go-task)，
然后通过源码安装，代替从 [下载页面](https://github.com/go-task/task/releases) 下载的二进制文件：

```cmd
yay -S go-task
```

这种方式由社区维护。

### Fedora

在 Fedora 上，可以使用 `dnf` 通过官方仓库
[Fedora](https://packages.fedoraproject.org/pkgs/golang-github-task/go-task/) 来安装：

```cmd
sudo dnf install go-task
```

这种方式由社区维护。新版 Task 发布后，需要一段时间才能通过
[Fedora](https://packages.fedoraproject.org/pkgs/golang-github-task/go-task/) 安装。

### Nix

在 NixOS 或安装了 Nix 的系统上，可以通过 [nixpkgs](https://github.com/NixOS/nixpkgs) 来安装：

```cmd
nix-env -iA nixpkgs.go-task
```

这种方式由社区维护。新版 Task 发布后，需要一段时间才能通过
[nixpkgs](https://github.com/NixOS/nixpkgs) 安装。

### npm

你还可以通过 Node 和 npm 的[包](https://www.npmjs.com/package/@go-task/cli) 来安装 Task：

```bash
npm install -g @go-task/cli
```

## 获取二进制文件

### 二进制文件

通过 [Github 下载页面][releases] 下载二进制文件，然后添加到 `$PATH` 路径。

还支持 DEB 和 RPM 包。

`task_checksums.txt` 文件包含每一个文件的 SHA-256 摘要。

### 安装脚本

我们的 [安装脚本][installscript] 在某些情况下也非常有用，
比如在 CI 中。多谢 [GoDownloader][godownloader] 帮助生成安装脚本。

默认情况下，安装脚本会安装到工作目录的相对路径 `./bin` 下：

```bash
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d
```

通过 `-b` 参数可以自定义安装目录，在 Linux 中当前用户安装一般会选择 `~/.local/bin` 或 `~/bin`，
全局用户安装会选择 `/usr/local/bin`：

```bash
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin
```

:::caution

在 macOS 和 Windows 中，`~/.local/bin` 和 `~/bin` 目录
默认是没有添加到 `$PATH` 当中的。

:::

### GitHub Actions

如果想在 GitHub Actions 中安装 Task，
可以尝试 Arduino 团队的 [action](https://github.com/arduino/setup-task)：

```yaml
- name: Install Task
  uses: arduino/setup-task@v1
```

这种方式由社区维护。

## 源代码安装

### Go Modules

首先，确保已经正确安装了 [Go][go]。

然后全局安装最新版本，只要这样：

```bash
go install github.com/go-task/task/v3/cmd/task@latest
```

或者安装到指定目录：

```bash
env GOBIN=/bin go install github.com/go-task/task/v3/cmd/task@latest
```

如果用的是 Go 1.15 或更早版本，这样安装：

```bash
env GO111MODULE=on go get -u github.com/go-task/task/v3/cmd/task@latest
```

:::tip

CI 环境中，我们建议使用 [安装脚本](#get-the-binary)，
更快捷更稳定，因为它只是下载了最新的发布版二进制文件。

:::

## 安装自动补全

下载符合你的 shell 的补全文件（快捷命令）。

[Task 仓库全部补全文件](https://github.com/go-task/task/tree/master/completion)

### Bash

首先，确认你通过包管理安装了 bash-completion。

给文件添加执行权限：

```
chmod +x path/to/task.bash
```

然后在 `~/.bash_profile` 文件中添加：

```shell
source path/to/task.bash
```

### ZSH

把 `_task` 文件放到你的 `$FPATH` 路径当中：

```shell
mv path/to/_task /usr/local/share/zsh/site-functions/_task
```

在 `~/.zshrc` 文件中添加：

```shell
autoload -U compinit
compinit -i
```

推荐使用 ZSH 5.7 或更高版本。

### Fish

移动 `task.fish` 补全脚本：

```shell
mv path/to/task.fish ~/.config/fish/completions/task.fish
```

### PowerShell

打开配置文件：

```
mkdir -Path (Split-Path -Parent $profile) -ErrorAction SilentlyContinue
notepad $profile
```

添加内容：

```shell
Invoke-Expression -Command path/to/task.ps1
```

[go]: https://golang.org/
[snapcraft]: https://snapcraft.io/task
[homebrew]: https://brew.sh/
[installscript]: https://github.com/go-task/task/blob/master/install-task.sh
[releases]: https://github.com/go-task/task/releases
[godownloader]: https://github.com/goreleaser/godownloader
[choco]: https://chocolatey.org/
[scoop]: https://scoop.sh/
