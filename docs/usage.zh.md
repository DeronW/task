# 用法

## 开始

在项目根目录创建 `Taskfile.yml` 文件。
`cmds` 属性应当包含 task 要执行的命令。
下面的例子定义了一个编译 Go 的应用和一个用 [Minify][minify] 压缩合并 CSS 的应用。

```yaml
version: "3"

tasks:
  build:
    cmds:
      - go build -v -i main.go

  assets:
    cmds:
      - minify -o public/style.css src/css
```

执行 task 也很简单:

```bash
task assets build
```

Task 采用了 [mvdan.cc/sh](https://mvdan.cc/sh/) ，
一个 Go 原生的 sh 编译器。
所以你的 sh/bash 命令甚至能在 windows 上通用。
但要注意使用的命令必须是操作系统支持的。

如果不传 task 的名字，默认会调用 "default"。

## 配置文件的命名

Task 会按顺序寻找配置文件:

- Taskfile.yml
- Taskfile.yaml
- Taskfile.dist.yml
- Taskfile.dist.yaml

允许 `.dist` 扩展文件是为了可以在项目中提交一个（`.dist`）配置文件，
同时允许每个开发者使用自定义的 `Taskfile.yml` 文件进行覆盖
（自定义文件应该在`.gitignore`中标记忽略）

## 环境变量

### Task

你可以使用 `env` 给每个 task 设置自定义环境变量:

```yaml
version: "3"

tasks:
  greet:
    cmds:
      - echo $GREETING
    env:
      GREETING: Hey, there!
```

另外，设置全局环境变量可以给所有 task 使用:

```yaml
version: "3"

env:
  GREETING: Hey, there!

tasks:
  greet:
    cmds:
      - echo $GREETING
```

> NOTE:
>
> `env` 还可以使用 shell 命令的结果，就像变量那样，
> 更多细节查看 [Variables](#variables) 章节。

### .env 文件

通过 `dotenv:` 可以让 Task 引用类似 `.env` 的文件：

title=".env"

```bash
KEYNAME=VALUE
```

title="testing/.env"

```bash
ENDPOINT=testing.com
```

title="Taskfile.yml"

```yaml
version: "3"

env:
  ENV: testing

dotenv: [".env", "{{.ENV}}/.env.", "{{.HOME}}/.env"]

tasks:
  greet:
    cmds:
      - echo "Using $KEYNAME and endpoint $ENDPOINT"
```

## 引用其它 Taskfile 文件

如果想要在不同项目复用 (Taskfiles)，可以使用引用机制，
设置 `includes` 属性：

```yaml
version: "3"

includes:
  docs: ./documentation # will look for ./documentation/Taskfile.yml
  docker: ./DockerTasks.yml
```

调用引用文件的 task 需要添加前缀使用。比如，使用 `task docs:serve`
来调用 `documentation/Taskfile.yml` 文件中的 `serve` 任务，
或使用 `task docker:build` 来调用 `DockerTasks.yml` 文件中的 `build` 任务。

相对路径的解析是相对 Taskfile 所在的目录。

### 依赖操作系统的 Taskfile

在 `version: '2'` 中，task 会自动尝试引入 `Taskfile_{{OS}}.yml` 文件
（例如`Taskfile_windows.yml`, `Taskfile_linux.yml` 或
`Taskfile_darwin.yml`）。但是因为过于隐晦，在版本 3 中被移除了，
在版本 3 可以通过明确的引用来实现类似功能:

```yaml
version: "3"

includes:
  build: ./Taskfile_{{OS}}.yml
```

### 被引用 Taskfile 的目录

默认情况下，引用文件中的 task 也是在当前路径执行的，
尽管被引用文件可能在另一个目录下，
但是可以强制指定引用文件的执行目录:

```yaml
version: "3"

includes:
  docs:
    taskfile: ./docs/Taskfile.yml
    dir: ./docs
```

> NOTE:
> 被引用文件必须使用与主 Taskfile 的版本相同。

### 可选引用

引用被标记为可选以后，如果文件不存在，Task 会继续执行其它命令。

```yaml
version: "3"

includes:
  tests:
    taskfile: ./tests/Taskfile.yml
    optional: true

tasks:
  greet:
    cmds:
      - echo "This command can still be successfully executed if ./tests/Taskfile.yml does not exist"
```

### 内部引用

引用被标记为内部以后，引用文件中的 task 也都被标记为内部 task
(查看 [内部任务](#内部任务) 内容)。
它的用途是引用工具类型的 task，同时避免任务被用户直接调用。

```yaml
version: "3"

includes:
  tests:
    taskfile: ./taskfiles/Utils.yml
    internal: true
```

### 引入文件中的变量

引用 Taskfile 文件时可以设置变量，
它可以在复用 Taskfile 时进行设置，即使多次引用相同文件:

```yaml
version: "3"

includes:
  backend:
    taskfile: ./taskfiles/Docker.yml
    vars:
      DOCKER_IMAGE: backend_image

  frontend:
    taskfile: ./taskfiles/Docker.yml
    vars:
      DOCKER_IMAGE: frontend_image
```

### 命名空间的别名

引用 Taskfile 后，可以给命名空间设置 `aliases` 列表。
工作方式与 [Task 别名](#task-别名) 相同，
并且共同使用是可以创造简短易用的命令。

```yaml
version: "3"

includes:
  generate:
    taskfile: ./taskfiles/Generate.yml
    aliases: [gen]
```

> NOTE:
> 引入文件中的变量优先级高于主文件！如果想要覆盖引入文件中的变量，
> 用 [default function](https://go-task.github.io/slim-sprig/defaults.html):
>
> `MY_VAR: '{{.MY_VAR | default "my-default-value"}}'`

## 内部任务

内部任务是不能直接调用的任务。它们不会出现在 `task --list|--list-all` 的列表中。
其它任务可以正常调用内部任务。这种方式有助于创建可复用、类函数、不会直接执行的任务。

```yaml
version: "3"

tasks:
  build-image-1:
    cmds:
      - task: build-image
        vars:
          DOCKER_IMAGE: image-1

  build-image:
    internal: true
    cmds:
      - docker build -t {{.DOCKER_IMAGE}} .
```

## 任务目录

默认情况下，task 会在 Taskfile 所在目录执行任务。
需要变更执行目录时，可以设置 `dir`:

```yaml
version: "3"

tasks:
  serve:
    dir: public/www
    cmds:
      # run http server
      - caddy
```

如果目录不存在，`task` 会自动创建。

## 任务依赖

> 依赖是并发执行的，所以依赖中的任务无法互相依赖。
> 如果要按顺序执行，查看 [调用其它任务](#调用其它任务)。

有些 task 可能依赖其它 task。只要在 `deps` 中标记，
就可以自动提前执行这些依赖:

```yaml
version: "3"

tasks:
  build:
    deps: [assets]
    cmds:
      - go build -v -i main.go

  assets:
    cmds:
      - minify -o public/style.css src/css
```

上面的例子中，如果运行 `task build`，那么 `assets` 总是会在 `build` 前执行。

task 可以只包含依赖，不包含命令，这样可以对任务进行聚合:

```yaml
version: "3"

tasks:
  assets:
    deps: [js, css]

  js:
    cmds:
      - minify -o public/script.js src/js

  css:
    cmds:
      - minify -o public/style.css src/css
```

有多个依赖时，会并发执行来提高效率。

> NOTE:
> 还可以通过命令行参数 `--parallel` (简写 `-p`) 来并行执行任务。
> 例如: `task --parallel js css`

如果想要给依赖任务传递参数，可以采取 [调用其它任务](#调用其它任务) 中的方法:

```yaml
version: "3"

tasks:
  default:
    deps:
      - task: echo_sth
        vars: { TEXT: "before 1" }
      - task: echo_sth
        vars: { TEXT: "before 2" }
    cmds:
      - echo "after"

  echo_sth:
    cmds:
      - echo {{.TEXT}}
```

## 调用其它任务

存在多个依赖时，这些依赖任务会并发执行。这样任务执行的会更快。
但有时下，需要按顺序执行 task。这种情况下可以使用下面的语法：

```yaml
version: "3"

tasks:
  main-task:
    cmds:
      - task: task-to-be-called
      - task: another-task
      - echo "Both done"

  task-to-be-called:
    cmds:
      - echo "Task to be called"

  another-task:
    cmds:
      - echo "Another task"
```

覆盖调用 task 中的变量，只需要设置 `vars` 属性:

```yaml
version: "3"

tasks:
  greet:
    vars:
      RECIPIENT: '{{default "World" .RECIPIENT}}'
    cmds:
      - echo "Hello, {{.RECIPIENT}}!"

  greet-pessimistically:
    cmds:
      - task: greet
        vars: { RECIPIENT: "Cruel World" }
```

上面的语法也可以用在 `deps` 中。

> NOTE
> 注意：如果在引用文件中调用主文件中的 task，需要添加 `:` 前缀，
> 例如: `task: :task-name`。

## 减少非必要工作

### 文件指纹

如果 task 有产出物，那么可以标记源文件和生成文件关系，
这样 Task 就可以减少非必要的任务。

```yaml
version: "3"

tasks:
  build:
    deps: [js, css]
    cmds:
      - go build -v -i main.go

  js:
    cmds:
      - minify -o public/script.js src/js
    sources:
      - src/js/**/*.js
    generates:
      - public/script.js

  css:
    cmds:
      - minify -o public/style.css src/css
    sources:
      - src/css/**/*.css
    generates:
      - public/style.css
```

`sources` 和 `generates` 可以配置具体文件，可以以使用匹配模式。设置后，
Task 会根据源文件的 checksum 来确定是否需要执行当前任务。如果不需要执行，
则会输出像 `Task "js" is up to date` 这样的信息。

对比文件的方法如果想用时间戳而不是 checksum，
只需要把 `method` 属性设置为 `timestamp`。

```yaml
version: "3"

tasks:
  build:
    cmds:
      - go build .
    sources:
      - ./*.go
    generates:
      - app{{exeExt}}
    method: timestamp
```

更复杂的情况可以使用 `status` 关键字。甚至可以多个进行组合使用。在文档 [程序判断 task 是否已更新](#程序判断-task-是否已更新) 中查看例子。

> NOTE:
> 默认情况，task 在本地项目的 `.task` 目录保存 checksums 值。
> 一般都会在 `.gitignore`（或类似配置）中忽略掉这个目录。
> （如果 task 中有生成代码的功能，那么提交这个目录似乎也有些道理）。
>
> 如果想要在其它目录保存这些文件，可以配置环境变量 `TASK_TEMP_DIR`。
> 可以使用相对路径，比如 `tmp/task`，相对项目根目录，
> 也可以用绝对路径、用户目录路径，比如 `/tmp/.task`
> 或 `~/.task`（每个项目单独创建子目录）。
>
> ```bash
> export TASK_TEMP_DIR='~/.task'
> ```

> NOTE:
> 每个 task 只有一个 `sources` 的 checksum。如果需要根据输入变量来区分 task，
> 就需要把变量作为 task 的一个标签，这样就会被当做一个不同的 task。
>
> 这种方式用于在不同入参下触发 task，直到 resource 发生的变化。
> 例如，你想让 resource 依赖某个变量，或者虽然 resource 没变，
> 但参数变化时依然重新执行 task。

> NOTE:
> 配置为 `none` 会跳过校验，每次都会执行 task。

> NOTE:
> 想要 `checksum`（默认）方法生效，只需要配置 source 文件，
> 但如果想用 `timestamp` 方法，还需要通过 `generates` 来标记生成的文件。

### 程序判断 task 是否已更新

还有一种方法，你可以使用一系列 test 作为 `status`。如果正常返回（退出信号是 0），
那么任务就被当做已更新:

```yaml
version: "3"

tasks:
  generate-files:
    cmds:
      - mkdir directory
      - touch directory/file1.txt
      - touch directory/file2.txt
    # test existence of files
    status:
      - test -d directory
      - test -f directory/file1.txt
      - test -f directory/file2.txt
```

一般情况，你需要 `generates` 和 `sources` 一起配合使用 - 但如果
存在远程生成物（Docker 镜像，部署，持续发布），checksum 和 timestamp
就需要有处理远程文件的权限，或能够刷新 `.checksum` 指纹文件的外挂。

这两个特殊变量 `{{.CHECKSUM}}` 和 `{{.TIMESTAMP}}` 可以作为 `status` 的
插值，具体取决于生成 source 指纹的方法。只有 `source` 块才能生成指纹。

注意， `{{.TIMESTAMP}}` 变量在 Go 语音 `time.Time` 结构体，
可以被任意 `time.Time` 的格式化方法。

查看 [the Go Time documentation](https://golang.org/pkg/time/) 了解更多信息。

可以使用 `--force` 或 `-f` 来强制 task 执行。

同时，`task --status [tasks]...` 会返回非 0 状态码，只要有任何 task 没有更新到最新。

`status` 可以与 [文件指纹](#文件指纹) 配合使用，
来判断一个 task 是否根据 source/generated 变化或程序检查失败的条件来执行:

```yaml
version: "3"

tasks:
  build:prod:
    desc: Build for production usage.
    cmds:
      - composer install
    # Run this task if source files changes.
    sources:
      - composer.json
      - composer.lock
    generates:
      - ./vendor/composer/installed.json
      - ./vendor/autoload.php
    # But also run the task if the last build was not a production build.
    status:
      - grep -q '"dev": false' ./vendor/composer/installed.json
```

### 程序检查任务执行条件

相比 `status` 检查，`preconditions` 是逻辑上相反的检查。
如果你需要前提条件为 _true_ ，可以使用 `preconditions` 字段。
`preconditions` 与 `status` 用法类似，
不仅都支持 `sh` 表达式，并且表达式返回值都应该是 0。

```yaml
version: "3"

tasks:
  generate-files:
    cmds:
      - mkdir directory
      - touch directory/file1.txt
      - touch directory/file2.txt
    # test existence of files
    preconditions:
      - test -f .env
      - sh: "[ 1 = 0 ]"
        msg: "One doesn't equal Zero, Halting"
```

前置条件可以设置错误信息，提示用户下一步操作，只要配置 `msg` 字段。

如果某个 task 依赖了带有前置条件的子 task，并且前置条件并未满足，则 task 会失败。
注意，前置条件失败的 task 不会执行，除非使用了 `--force` 标记。

与 `status` 判断 task 是最新状态时会跳过并继续执行不同，
`precondition` 失败会导致 task 失败，以及所有依赖它的 task 都会失败。

```yaml
version: "3"

tasks:
  task-will-fail:
    preconditions:
      - sh: "exit 1"

  task-will-also-fail:
    deps:
      - task-will-fail

  task-will-still-fail:
    cmds:
      - task: task-will-fail
      - echo "I will not run"
```

### 限制 task 执行次数

如果一个 task 被多个 `cmds` 调用，或被多个 `deps` 依赖，
那么可以通过 `run` 来控制执行过程。
`run` 也可以设置为 Taskfile 的根属性，设置全局默认行为。

`run` 支持参数:

- `always` (默认) 每次调用都会执行，不论之前是否被调用过
- `once` 只掉一次，不论之后是否再次被调用
- `when_changed` 每组变量仅调用一次

```yaml
version: "3"

tasks:
  default:
    cmds:
      - task: generate-file
        vars: { CONTENT: "1" }
      - task: generate-file
        vars: { CONTENT: "2" }
      - task: generate-file
        vars: { CONTENT: "2" }

  generate-file:
    run: when_changed
    deps:
      - install-deps
    cmds:
      - echo {{.CONTENT}}

  install-deps:
    run: once
    cmds:
      - sleep 5 # long operation like installing packages
```

## 变量

使用变量进行插值时，Task 会按照以下优先级进行查找（高优先级在前）:

- task 内部定义的变量
- 被其它 task 调用时传入的变量(查看 [调用其它任务](#调用其它任务))
- [引用 Taskfile](#引用其它-Taskfile-文件)中的变量 (当引入 task 时)
- [主文件中的变量](#引入文件中的变量)中的变量 (当引入 task 时)
- 全局变量 (在 Taskfile 的 `vars:` 中声明)
- 系统环境变量

利用环境变量传递参数的例子:

```bash
$ TASK_VARIABLE=a-value task do-something
```

> NOTE:
> 变量 `.TASK` 始终代表 task 的名字。

因为有些 shell 不支持上面的语法（Windows）task 还支持环境变量不在开头的命令。

```bash
$ task write-file FILE=file.txt "CONTENT=Hello, World!" print "MESSAGE=All done!"
```

声明内部变量例子：

```yaml
version: "3"

tasks:
  print-var:
    cmds:
      - echo "{{.VAR}}"
    vars:
      VAR: Hello!
```

在 `Taskfile.yml` 中声明全局变量例子：

```yaml
version: "3"

vars:
  GREETING: Hello from Taskfile!

tasks:
  greet:
    cmds:
      - echo "{{.GREETING}}"
```

### 动态变量

下面语法 (使用 `sh:` 属性) 声明动态变量。
命令输出结果会赋给变量。如果输出结果是多行，最后的空行会被忽略掉。

```yaml
version: "3"

tasks:
  build:
    cmds:
      - go build -ldflags="-X main.Version={{.GIT_COMMIT}}" main.go
    vars:
      GIT_COMMIT:
        sh: git log -n 1 --format=%h
```

这种方式适用于所有类型的变量。

## 透传命令行参数

如果命令行中使用了 `--` ，它后面的参数会被赋给内置变量 `.CLI_ARGS` 。
可用于透传参数给另一个命令。

下面是运行 `yarn install` 的例子。

```bash
$ task yarn -- install
```

```yaml
version: "3"

tasks:
  yarn:
    cmds:
      - yarn {{.CLI_ARGS}}
```

## 使用 `defer` 做任务清理

使用 `defer` 关键字可以在 task 结束时进行清理。
与放在最后一行的命令之间的差别是，即使 task 失败了，它也会执行。

下面例子中，即使第三行的命令执行失败，`rm -rf tmpdir/` 依然会执行：

```yaml
version: "3"

tasks:
  default:
    cmds:
      - mkdir -p tmpdir/
      - defer: rm -rf tmpdir/
      - echo 'Do work on tmpdir/'
```

使用其它 task 作为清理任务的命令时，可以这样：

```yaml
version: "3"

tasks:
  default:
    cmds:
      - mkdir -p tmpdir/
      - defer: { task: cleanup }
      - echo 'Do work on tmpdir/'

  cleanup: rm -rf tmpdir/
```

> NOTE:
> 根据 [Go's 的 `defer` 工作方式](https://go.dev/tour/flowcontrol/13)，
> 被推迟的任务是按相反的顺序执行的，如果你定义了多个清理任务。

## Go 的模板引擎

Task 在执行命令前会先用 [Go 模板引擎][gotemplate] 进行解析。
变量也支持点语法 (`.VARNAME`)。

所有 [slim-sprig lib](https://go-task.github.io/slim-sprig/)
中的方法都支持。下面是按照指定格式显示时间的例子：

```yaml
version: "3"

tasks:
  print-date:
    cmds:
      - echo {{now | date "2006-01-02"}}
```

Task 还支持下面这些方法：

- `OS`: 当前操作系统，返回值可能是： "windows", "linux",
  "darwin" (macOS) 和 "freebsd"
- `ARCH`: 返回 Task 的编译架构："386", "amd64", "arm"
  或 "s390x"
- `splitLines`: 拆分 Unix (\n) 和 Windows (\r\n) 风格的换行
- `catLines`: 将 Unix (\n) 和 Windows (\r\n) 的换行替换成空格
- `toSlash`: 不操作 Unix, 但在 Windows 上将路径符 `\` 转换为 `/`
- `fromSlash`: 与 `toSlash` 相反，不操作 Unix, 但在 Windows 上将路径符 `/` 转换为 `\`
- `exeExt`: 返回当前操作系统执行文件结尾类型
  (Windows 是 `".exe"`, 其它的是 `""`)
- `shellQuote`: 给字符串加引号，以完成 shell 脚本运行，
  Task 用 [Go 方法](https://pkg.go.dev/mvdan.cc/sh/v3@v3.4.0/syntax#Quote)
  进行转义。采用 Bash 语法。

例子:

```yaml
version: "3"

tasks:
  print-os:
    cmds:
      - echo '{{OS}} {{ARCH}}'
      - echo '{{if eq OS "windows"}}windows-command{{else}}unix-command{{end}}'
      # This will be path/to/file on Unix but path\to\file on Windows
      - echo '{{fromSlash "path/to/file"}}'
  enumerated-file:
    vars:
      CONTENT: |
        foo
        bar
    cmds:
      - |
        cat << EOF > output.txt
        {{range $i, $line := .CONTENT | splitLines -}}
        {{printf "%3d" $i}}: {{$line}}
        {{end}}EOF
```

## 帮助

执行 `task --list` (或 `task -l`) 列出全部任务和描述。
下面的 Taskfile:

```yaml
version: "3"

tasks:
  build:
    desc: Build the go binary.
    cmds:
      - go build -v -i main.go

  test:
    desc: Run all the go tests.
    cmds:
      - go test -race ./...

  js:
    cmds:
      - minify -o public/script.js src/js

  css:
    cmds:
      - minify -o public/style.css src/css
```

输出结果：

```bash
* build:   Build the go binary.
* test:    Run all the go tests.
```

查看全部 task，可以使用 `--list-all` (简写为 `-a`)。

## 显示 task 总览

执行 `task --summary task-name` 可以显示 task 的总览。

下面的 Taskfile：

```yaml
version: "3"

tasks:
  release:
    deps: [build]
    summary: |
      Release your project to github

      It will build your project before starting the release.
      Please make sure that you have set GITHUB_TOKEN before starting.
    cmds:
      - your-release-tool

  build:
    cmds:
      - your-build-tool
```

执行 `task --summary release` 会输出：

```
task: release

Release your project to github

It will build your project before starting the release.
Please make sure that you have set GITHUB_TOKEN before starting.

dependencies:
 - build

commands:
 - your-release-tool
```

如果没有总览，则会使用描述代替。如果总览和描述都没有，会显示一个警告。

注意：_显示总览并不会执行命令_

## Task 别名

别名是 task 的另一个名字，方便引用那些又长又难写的名字。
你可以在命令行使用，当 [调用其它任务](#调用其它任务) 或 [引用其它 Taskfile 文件](#引用其它-Taskfile-文件) 中有别名时，可以与 [命名空间](#命名空间的别名) 一同使用。

```yaml
version: "3"

tasks:
  generate:
    aliases: [gen]
    cmds:
      - task: gen-mocks

  generate-mocks:
    aliases: [gen-mocks]
    cmds:
      - echo "generating..."
```

## 覆盖 task 名字

有时想要在概述、更新等信息中覆写 task 名字。
可以设置一个`label:`，它可以使用变量：

```yaml
version: "3"

tasks:
  default:
    - task: print
      vars:
        MESSAGE: hello
    - task: print
      vars:
        MESSAGE: world

  print:
    label: "print-{{.MESSAGE}}"
    cmds:
      - echo "{{.MESSAGE}}"
```

## 安静模式

安静模式下 Task 执行前不会再打印执行命令。
如下 Taskfile：

```yaml
version: "3"

tasks:
  echo:
    cmds:
      - echo "Print something"
```

正常会输出：

```sh
echo "Print something"
Print something
```

安静模式下，则会输出：

```sh
Print something
```

有 4 种方法开启安静模式：

- 命令级别：

```yaml
version: "3"

tasks:
  echo:
    cmds:
      - cmd: echo "Print something"
        silent: true
```

- task 级别:

```yaml
version: "3"

tasks:
  echo:
    cmds:
      - echo "Print something"
    silent: true
```

- 在 Taskfile 全局:

```yaml
version: "3"

silent: true

tasks:
  echo:
    cmds:
      - echo "Print something"
```

- 全局执行参数 `--silent` 或 `-s`

如果只想代替标准输出，可以将命令输出重定向到 `/dev/null`：

```yaml
version: "3"

tasks:
  echo:
    cmds:
      - echo "This will print nothing" > /dev/null
```

## Dry run 模式

Dry run 模式 (`--dry`) 会逐条编译、输出每个 task 的命令，
但不会真的执行，可用于调试 Taskfile。

## 忽略错误

可以配置选项忽略命令执行过程中的错误。
参考下面 Taskfile：

```yaml
version: "3"

tasks:
  echo:
    cmds:
      - exit 1
      - echo "Hello World"
```

Task 执行 `exit 1` 后会终止，因为状态码 `1` 表示 `EXIT_FAILURE`。
但是配置 `ignore_error` 可以忽略错误继续执行：

```yaml
version: "3"

tasks:
  echo:
    cmds:
      - cmd: exit 1
        ignore_error: true
      - echo "Hello World"
```

`ignore_error` 也可以在 task 中设置，表示忽略所有命令的错误。
但是注意，这个配置不会影响 `deps` 或 `cmds` 中的 task。

## 输出模式

默认情况，Task 只会将执行命令的 STDOUT 和 STDERR 实时重定向到终端。
这有助于实时反馈和记录命令执行，但是有多个命令同时打印大量日志时就变得很凌乱。

为了方便定制，有三种输出模式可选：

- `interleaved` (默认)
- `group`
- `prefixed`

选择不同模式，只需要在 Taskfile 根属性上进行设置：

```yaml
version: "3"

output: "group"

tasks:
  # ...
```

`group` 模式会在命令结束后输出全部日志，这种模式下反馈会有延迟时间。

使用`group`输出，可以提供一个可选消息模板，在消息分组的开始和结尾显示。
可用于 CI 系统对一个任务的输出进行分组，
比如与 [GitHub Actions' `::group::` command](https://docs.github.com/en/actions/learn-github-actions/workflow-commands-for-github-actions#grouping-log-lines)
或 [Azure Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?expand=1&view=azure-devops&tabs=bash#formatting-commands) 一同使用。

```yaml
version: "3"

output:
  group:
    begin: "::group::{{.TASK}}"
    end: "::endgroup::"

tasks:
  default:
    cmds:
      - echo 'Hello, World!'
    silent: true
```

```bash
$ task default
::group::default
Hello, World!
::endgroup::
```

`prefix` 选项会给每一行输出添加一个 `[task-name] ` 前缀，
也可以通过 `prefix:` 自定义前缀：

```yaml
version: "3"

output: prefixed

tasks:
  default:
    deps:
      - task: print
        vars: { TEXT: foo }
      - task: print
        vars: { TEXT: bar }
      - task: print
        vars: { TEXT: baz }

  print:
    cmds:
      - echo "{{.TEXT}}"
    prefix: "print-{{.TEXT}}"
    silent: true
```

```bash
$ task default
[print-foo] foo
[print-bar] bar
[print-baz] baz
```

> NOTE:
> `output` 选项也可以通过命令行参数`--output` 或 `-o` 指定。

## 交互式命令行应用

Task 执行包含交互式的命令时会出现奇怪的结果，
尤其当 [输出模式](#输出模式) 设置的不是 `interleaved` （默认），
或者当交互式应用与其它 task 并发执行时。

设置 `interactive: true` 让 Task 知道这是个交互式应用，并自动优化：

```yaml
version: "3"

tasks:
  default:
    cmds:
      - vim my-file.txt
    interactive: true
```

如果在运行交互式应用时遇到其它问题，请提交 issue。

## 短 task 语法

从 v3 版本开始，如果有默认值就可以使用断语法
（比如，`env:`, `vars:`, `desc:`, `silent:` 等等）：

```yaml
version: "3"

tasks:
  build: go build -v -o ./app{{exeExt}} .

  run:
    - task: build
    - ./app{{exeExt}} -h localhost -p 8080
```

## 文件监控

使用 `--watch` 或 `-w` 参数可以监控文件变化，然后重新执行 task。
这需要配置 `sources` 属性，task 才知道监控哪些文件。

默认监控的时间间隔是 5 秒，即可以通过 Taskfile 中
根属性 `interval: '500ms'` 设置，也可以通过命令行
参数 `--interval=500ms` 设置。

[gotemplate]: https://golang.org/pkg/text/template/
[minify]: https://github.com/tdewolff/minify/tree/master/cmd/minify
