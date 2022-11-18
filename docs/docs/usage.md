---
slug: /usage/
sidebar_position: 3
---

# 用法

## 开始

在项目根目录创建 `Taskfile.yml` 文件。`cmds` 属性应当包含 task 要执行的命令。
下面的例子定义了一个 Go 应用和一个用 [Minify][minify] 压缩合并 CSS 应用。

```yaml
version: '3'

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

Task 采用了 [mvdan.cc/sh](https://mvdan.cc/sh/) ，一个 Go 的 sh 编译器。所以你的 sh/bash 命令甚至能用到 windows 上。
但要注意使用的命令需要操作系统支持。

如果省略了 task 的名字，默认会用 "default"。

## 配置文件名称

Task 会按顺序寻找配置文件:

- Taskfile.yml
- Taskfile.yaml
- Taskfile.dist.yml
- Taskfile.dist.yaml

使用 `.dist` 扩展文件是为了提交一个生公共版本（`.dist`）同时允许每个开发者使用自定义的 `Taskfile.yml` 文件进行覆盖（这个文件应该在`.gitignore`中标记忽略）

## 环境变量 Environment variables

### Task

你可以使用 `env` 给每个 task 设置自定义环境变量:

```yaml
version: '3'

tasks:
  greet:
    cmds:
      - echo $GREETING
    env:
      GREETING: Hey, there!
```

另外，设置的全局环境变量可以在所有 task 中使用:

```yaml
version: '3'

env:
  GREETING: Hey, there!

tasks:
  greet:
    cmds:
      - echo $GREETING
```

:::说明

`env` 还能扩展到使用 shell 命令的结果，就像变量那样，更多细节查看 [Variables](#variables) 章节。

### 环境变量文件 .env

通过 `dotenv:` 可以让 Task 引用 `.env` 文件:

```bash title=".env"
KEYNAME=VALUE
```

```bash title="testing/.env"
ENDPOINT=testing.com
```

```yaml title="Taskfile.yml"
version: '3'

env:
  ENV: testing

dotenv: ['.env', '{{.ENV}}/.env.', '{{.HOME}}/.env']

tasks:
  greet:
    cmds:
      - echo "Using $KEYNAME and endpoint $ENDPOINT"
```

## 引用 Taskfile

如果想要在不同项目复用 (Taskfiles)，可以使用引用机制，设置 `includes` 关键字:

```yaml
version: '3'

includes:
  docs: ./documentation # will look for ./documentation/Taskfile.yml
  docker: ./DockerTasks.yml
```

Taskfiles 中定义的 task 都可以通过引用时设置的命名空间名字来使用。比如，使用命令 `task docs:serve` 来调用 `documentation/Taskfile.yml` 文件中的  `serve` 任务，
或使用  `task docker:build`  来调用`DockerTasks.yml` 文件中的 `build` 任务。

相对路径的解析是相对包含 Taskfile 的目录。

### 特定操作系统的 Taskfile

在 `version: '2'` 中，task 尝试自动引入 `Taskfile_{{OS}}.yml` 文件（例如`Taskfile_windows.yml`, `Taskfile_linux.yml` 或
`Taskfile_darwin.yml`）。但是因为过于隐晦，在版本 3 中被移除了，但版本 3 中依然可以通过明确的引用来实现类似功能:

```yaml
version: '3'

includes:
  build: ./Taskfile_{{OS}}.yml
```

### 被引用 Taskfile 的目录

默认情况下，引用文件中的 task 也是在当前路径执行的，尽管被引用文件可能在另一个目录下，
但是可以强制指定引用文件的执行目录:

```yaml
version: '3'

includes:
  docs:
    taskfile: ./docs/Taskfile.yml
    dir: ./docs
```

:::说明

被引用文件必须使用与主 Taskfile 相同的版本。

:::

### 可选引用 Optional includes

引用被标记为可选以后，即使文件不存在 Task 也可以继续执行其它命令。

```yaml
version: '3'

includes:
  tests:
    taskfile: ./tests/Taskfile.yml
    optional: true

tasks:
  greet:
    cmds:
      - echo "This command can still be successfully executed if ./tests/Taskfile.yml does not exist"
```

### 内部引用 Internal includes

引用被标记为内部以后，引用文件中的 task 也都被标记为内部 task(查看 [Internal tasks](#internal-tasks) 内容)。
它的用途是引用工具类型的 task 时可以避免被用户直接调用。

```yaml
version: '3'

includes:
  tests:
    taskfile: ./taskfiles/Utils.yml
    internal: true
```

### 引入文件的变量 Vars of included Taskfiles

引用 Taskfile 文件时可以指定变量，它的用途是在复用 Taskfile 时可以进行调整，甚至多次引用:

```yaml
version: '3'

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

### 命名空间别名 Namespace aliases

引用 Taskfile 后，可以给 namespace 设置 `aliases` 列表。
工作方式与 [task aliases](#task-aliases) 相同并且可以共同使用创造简短易用的命令。

```yaml
version: '3'

includes:
  generate:
    taskfile: ./taskfiles/Generate.yml
    aliases: [gen]
```

:::说明

引入文件中的变量优先级高于主文件！如果想要覆盖引入文件中的变量，用 [default function](https://go-task.github.io/slim-sprig/defaults.html):           
`MY_VAR: '{{.MY_VAR | default "my-default-value"}}'`

:::

## 内部任务 Internal tasks

内部任务是不能被直接调用的任务。它们不会出现在 `task --list|--list-all` 的列表中。
其它任务可以正常调用内部任务。这种方式有助于创建可复用、类函数，但不需要命令行执行的任务。

```yaml
version: '3'

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

## 任务目录 Task directory

默认情况下，task 会在 Taskfile 所在目录执行任务。需要变更执行目录时，只需要设置 `dir`:

```yaml
version: '3'

tasks:
  serve:
    dir: public/www
    cmds:
      # run http server
      - caddy
```

如果目录不存在，`task` 会创建一个。

## 任务依赖 Task dependencies

> 依赖是并发执行的，所以依赖中的任务不能互相依赖。如果要按顺序执行，查看 [Calling Another Task](#calling-another-task) 章节。

你可能有些 task 是依赖其它任务的。只要在 `deps` 中标记，就可以自动提前执行依赖:

```yaml
version: '3'

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
version: '3'

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

:::提示

还可以通过命令行参数 `--parallel` (简写 `-p`) 来并行执行任务。例如 : `task --parallel js css`

:::

如果想要给依赖任务传递参数，可以采取与 [call another task](#calling-another-task) 相同的方法:

```yaml
version: '3'

tasks:
  default:
    deps:
      - task: echo_sth
        vars: {TEXT: "before 1"}
      - task: echo_sth
        vars: {TEXT: "before 2"}
    cmds:
      - echo "after"

  echo_sth:
    cmds:
      - echo {{.TEXT}}
```

## 调用其它任务 Calling another task

存在多个依赖时，这些依赖任务会并发执行。这样任务执行的会更快。但是，某些情况下，你需要按顺序调用任务。
这种情况下可以使用下面的语法

```yaml
version: '3'

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

覆盖调用任务重的变量，只需要设置 `vars` 属性:

```yaml
version: '3'

tasks:
  greet:
    vars:
      RECIPIENT: '{{default "World" .RECIPIENT}}'
    cmds:
      - echo "Hello, {{.RECIPIENT}}!"

  greet-pessimistically:
    cmds:
      - task: greet
        vars: {RECIPIENT: "Cruel World"}
```

上面的语法也可以用在 `deps` 中。

:::提示

注意：如果在引用文件中调用主文件中的任务，需要添加 `:` 前缀，例如:
`task: :task-name`。

:::

## 节省非必要工作 Prevent unnecessary work

### 本地生成文件指纹 By fingerprinting locally generated files and their sources

如果 task 有生成物，就可以标记源文件和生成文件关系，这样 Task 就不会执行不必要的任务。

```yaml
version: '3'

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

`sources` 和 `generates` 可以指定文件或采用匹配模式。设置后，
Task 会对比源文件的 checksum 来决定是否需要执行当前任务。如果不需要执行，
则会输出像 `Task "js" is up to date` 这样的信息。

对比文件的方法如果想用时间戳而不是 checksum，只需要把 `method` 属性设置为 `timestamp`。

```yaml
version: '3'

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

更复杂的情况可以使用 `status` 关键字。
甚至可以进行组合使用。查看文档 [status](#using-programmatic-checks-to-indicate-a-task-is-up-to-date) 了解更多细节。


:::说明

默认情况，task 在本地项目的 `.task` 目录保存 checksums 值。一般都会在 `.gitignore`（或类似配置） 
中忽略掉这个目录，来避免被提交。（如果 task 中有生成代码的功能，那么提交这个目录似乎也有些道理）。

如果想要在其它目录保存这些文件，可以设置 `TASK_TEMP_DIR` 环境变量。支持相对路径，比如 `tmp/task`，
相对项目目录，或绝对路径、用户目录路径，比如 `/tmp/.task` 或 `~/.task`（每个项目都会创建子目录）。

```bash
export TASK_TEMP_DIR='~/.task'
```

:::

:::说明

每个 task 都有保存一个 `sources` 的 checksum。如果需要根据输入变量来区分 task，就需要
把变量作为 task 的一个标签，这样就会被当做一个不同的 task。

这种方式用于在不同入参下触发 task，直到 resource 发生真正的变化。例如，你想让 resource 依赖
某个变量，或者虽然 resource 没变，但参数变化时依然重新执行 task。

:::

:::提示

`none` 方法跳过校验，每次都会执行 task。

:::

:::说明

想要 `checksum`（默认）方法生效，只需要配置 source 文件，但如果想用 `timestamp` 方法，
你还需要通过 `generates` 来标记生成的文件。

:::

### 程序判断 task 是否已更新 Using programmatic checks to indicate a task is up to date.

还有一种方法，你可以使用一系列测试作为 `status`。如果正常返回（退出信号是 0），那么任务就被当做已更新:

```yaml
version: '3'

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

一般情况，你需要 `generates` 需要 `sources` 一起配合使用 - 但如果存在远程生成物（Docker 镜像，部署，持续发布）
checksum 和 timestamp 就需要有处理远程文件的权限，或能够刷新 `.checksum` 指纹文件的外挂。

有两个特殊变量 `{{.CHECKSUM}}` 和 `{{.TIMESTAMP}}` 可以作为 `status` 命令的插值，具体取决于生成 source 指纹的方法。
只有 `source` 块才能生成指纹。

注意， `{{.TIMESTAMP}}` 变量在 Go 语音 `time.Time` 结构体，可以被相应的 `time.Time` 方法进行格式化。

查看 [the Go Time documentation](https://golang.org/pkg/time/) 了解更多信息。

可以使用 `--force` 或 `-f` 来强制 task 执行。

同时，`task --status [tasks]...` 会返回非 0 状态码，只要有任何 task 没有更新到最新。

`status` 可以与 [fingerprinting](#by-fingerprinting-locally-generated-files-and-their-sources) 配合使用，
来判断一个 task 是否根据 source/generated 变化或程序检查失败的条件来执行:

```yaml
version: '3'

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

### 程序检查任务执行条件 Using programmatic checks to cancel the execution of a task and its dependencies

相比 `status` 检查，`preconditions` 是逻辑上相反的检查。如果你需要前提条件为 _true_ ，
可以使用  `preconditions` 字段。 `preconditions` 与 `status` 用法类似，
不仅都支持 `sh` 表达式，并且表达式返回值都应该是 0。

```yaml
version: '3'

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

与 `status` 不同，如果 task 已经是最近状态，则会跳过然后继续执行，一个 `precondition` 会导致
一个 task 失败，顺带依赖当前 task 的 task 都会失败。

```yaml
version: '3'

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

### 限制 task 执行 Limiting when tasks run

如果一个 task 被多个 `cmds` 调用，或被多个 `deps` 依赖，那么可以通过 `run` 来控制执行过程。
`run` 也可以设置为 Taskfile 的根属性，设置全局默认执行行为。

`run` 支持参数:

 * `always` (默认) 每次调用都会执行，不论之前是否被调用过
 * `once` 只掉一次，不论之后是否再次被引用
 * `when_changed` 每组变量仅调用一次

```yaml
version: '3'

tasks:
  default:
    cmds:
      - task: generate-file
        vars: { CONTENT: '1' }
      - task: generate-file
        vars: { CONTENT: '2' }
      - task: generate-file
        vars: { CONTENT: '2' }

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

## 变量 Variables

使用变量进行插值时，Task 会按照以下优先级进行查找（高优先级在前）:

- task 内部定义的变量
- 被其它 task 调用时传入的变量(查看 [Calling another task](#calling-another-task))
- 引用文件中的变量 [included Taskfile](#including-other-taskfiles) (当引入 task 时)
- Variables of the [inclusion of the Taskfile](#vars-of-included-taskfiles) (when the task is included)
- 全局变量 (在 Taskfile 的 `vars:` 中声明)
- 系统环境变量

利用环境变量传递参数的例子:

```bash
$ TASK_VARIABLE=a-value task do-something
```

:::提示

变量 `.TASK` 始终代表 task 的名字。

:::

因为有些 shell 不支持上面的语法（Windows）task 还支持环境变量不在开头的命令。

```bash
$ task write-file FILE=file.txt "CONTENT=Hello, World!" print "MESSAGE=All done!"
```

Example of locally declared vars:

```yaml
version: '3'

tasks:
  print-var:
    cmds:
      - echo "{{.VAR}}"
    vars:
      VAR: Hello!
```

Example of global vars in a `Taskfile.yml`:

```yaml
version: '3'

vars:
  GREETING: Hello from Taskfile!

tasks:
  greet:
    cmds:
      - echo "{{.GREETING}}"
```

### Dynamic variables

The below syntax (`sh:` prop in a variable) is considered a dynamic variable.
The value will be treated as a command and the output assigned. If there are one
or more trailing newlines, the last newline will be trimmed.

```yaml
version: '3'

tasks:
  build:
    cmds:
      - go build -ldflags="-X main.Version={{.GIT_COMMIT}}" main.go
    vars:
      GIT_COMMIT:
        sh: git log -n 1 --format=%h
```

This works for all types of variables.

## Forwarding CLI arguments to commands

If `--` is given in the CLI, all following parameters are added to a
special `.CLI_ARGS` variable. This is useful to forward arguments to another
command.

The below example will run `yarn install`.

```bash
$ task yarn -- install
```

```yaml
version: '3'

tasks:
  yarn:
    cmds:
      - yarn {{.CLI_ARGS}}
```

## Doing task cleanup with `defer`

With the `defer` keyword, it's possible to schedule cleanup to be run once
the task finishes. The difference with just putting it as the last command is
that this command will run even when the task fails.

In the example below, `rm -rf tmpdir/` will run even if the third command fails:

```yaml
version: '3'

tasks:
  default:
    cmds:
      - mkdir -p tmpdir/
      - defer: rm -rf tmpdir/
      - echo 'Do work on tmpdir/'
```

If you want to move the cleanup command into another task, that is possible as
well:

```yaml
version: '3'

tasks:
  default:
    cmds:
      - mkdir -p tmpdir/
      - defer: { task: cleanup }
      - echo 'Do work on tmpdir/'

  cleanup: rm -rf tmpdir/
```

:::info

Due to the nature of how the
[Go's own `defer` work](https://go.dev/tour/flowcontrol/13), the deferred
commands are executed in the reverse order if you schedule multiple of them.

:::

## Go's template engine

Task parse commands as [Go's template engine][gotemplate] before executing
them. Variables are accessible through dot syntax (`.VARNAME`).

All functions by the Go's [slim-sprig lib](https://go-task.github.io/slim-sprig/)
are available. The following example gets the current date in a given format:

```yaml
version: '3'

tasks:
  print-date:
    cmds:
      - echo {{now | date "2006-01-02"}}
```

Task also adds the following functions:

- `OS`: Returns the operating system. Possible values are "windows", "linux",
  "darwin" (macOS) and "freebsd".
- `ARCH`: return the architecture Task was compiled to: "386", "amd64", "arm"
  or "s390x".
- `splitLines`: Splits Unix (\n) and Windows (\r\n) styled newlines.
- `catLines`: Replaces Unix (\n) and Windows (\r\n) styled newlines with a space.
- `toSlash`: Does nothing on Unix, but on Windows converts a string from `\`
  path format to `/`.
- `fromSlash`: Opposite of `toSlash`. Does nothing on Unix, but on Windows
  converts a string from `/` path format to `\`.
- `exeExt`: Returns the right executable extension for the current OS
  (`".exe"` for Windows, `""` for others).
- `shellQuote`: Quotes a string to make it safe for use in shell scripts.
  Task uses [this Go function](https://pkg.go.dev/mvdan.cc/sh/v3@v3.4.0/syntax#Quote)
  for this. The Bash dialect is assumed.

Example:

```yaml
version: '3'

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

## Help

Running `task --list` (or `task -l`) lists all tasks with a description.
The following Taskfile:

```yaml
version: '3'

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

would print the following output:

```bash
* build:   Build the go binary.
* test:    Run all the go tests.
```

If you want to see all tasks, there's a `--list-all` (alias `-a`) flag as well.

## Display summary of task

Running `task --summary task-name` will show a summary of a task.
The following Taskfile:

```yaml
version: '3'

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

with running ``task --summary release`` would print the following output:

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
If a summary is missing, the description will be printed.
If the task does not have a summary or a description, a warning is printed.

Please note: *showing the summary will not execute the command*.

## Task aliases

Aliases are alternative names for tasks. They can be used to make it easier and
quicker to run tasks with long or hard-to-type names. You can use them on the
command line, when [calling sub-tasks](#calling-another-task) in your Taskfile
and when [including tasks](#including-other-taskfiles) with aliases from another
Taskfile. They can also be used together with [namespace
aliases](#namespace-aliases).

```yaml
version: '3'

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

## Overriding task name

Sometimes you may want to override the task name printed on the summary, up-to-date
messages to STDOUT, etc. In this case, you can just set `label:`, which can also
be interpolated with variables:

```yaml
version: '3'

tasks:
  default:
    - task: print
      vars:
        MESSAGE: hello
    - task: print
      vars:
        MESSAGE: world

  print:
    label: 'print-{{.MESSAGE}}'
    cmds:
      - echo "{{.MESSAGE}}"
```

## Silent mode

Silent mode disables the echoing of commands before Task runs it.
For the following Taskfile:

```yaml
version: '3'

tasks:
  echo:
    cmds:
      - echo "Print something"
```

Normally this will be printed:

```sh
echo "Print something"
Print something
```

With silent mode on, the below will be printed instead:

```sh
Print something
```

There are four ways to enable silent mode:

* At command level:

```yaml
version: '3'

tasks:
  echo:
    cmds:
      - cmd: echo "Print something"
        silent: true
```

* At task level:

```yaml
version: '3'

tasks:
  echo:
    cmds:
      - echo "Print something"
    silent: true
```

* Globally at Taskfile level:

```yaml
version: '3'

silent: true

tasks:
  echo:
    cmds:
      - echo "Print something"
```

* Or globally with `--silent` or `-s` flag

If you want to suppress STDOUT instead, just redirect a command to `/dev/null`:

```yaml
version: '3'

tasks:
  echo:
    cmds:
      - echo "This will print nothing" > /dev/null
```

## Dry run mode

Dry run mode (`--dry`) compiles and steps through each task, printing the commands
that would be run without executing them. This is useful for debugging your Taskfiles.

## Ignore errors

You have the option to ignore errors during command execution.
Given the following Taskfile:

```yaml
version: '3'

tasks:
  echo:
    cmds:
      - exit 1
      - echo "Hello World"
```

Task will abort the execution after running `exit 1` because the status code `1` stands for `EXIT_FAILURE`.
However, it is possible to continue with execution using `ignore_error`:

```yaml
version: '3'

tasks:
  echo:
    cmds:
      - cmd: exit 1
        ignore_error: true
      - echo "Hello World"
```

`ignore_error` can also be set for a task, which means errors will be suppressed
for all commands. Nevertheless, keep in mind that this option will not propagate to other tasks
called either by `deps` or `cmds`!

## Output syntax

By default, Task just redirects the STDOUT and STDERR of the running commands
to the shell in real-time. This is good for having live feedback for logging
printed by commands, but the output can become messy if you have multiple
commands running simultaneously and printing lots of stuff.

To make this more customizable, there are currently three different output
options you can choose:

- `interleaved` (default)
- `group`
- `prefixed`

To choose another one, just set it to root in the Taskfile:

```yaml
version: '3'

output: 'group'

tasks:
  # ...
```

The `group` output will print the entire output of a command once after it
finishes, so you will not have live feedback for commands that take a long time
to run.

When using the `group` output, you can optionally provide a templated message
to print at the start and end of the group. This can be useful for instructing
CI systems to group all of the output for a given task, such as with
[GitHub Actions' `::group::` command](https://docs.github.com/en/actions/learn-github-actions/workflow-commands-for-github-actions#grouping-log-lines)
or [Azure Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?expand=1&view=azure-devops&tabs=bash#formatting-commands).

```yaml
version: '3'

output:
  group:
    begin: '::group::{{.TASK}}'
    end: '::endgroup::'

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

The `prefix` output will prefix every line printed by a command with
`[task-name] ` as the prefix, but you can customize the prefix for a command
with the `prefix:` attribute:

 ```yaml
version: '3'

output: prefixed

tasks:
  default:
    deps:
      - task: print
        vars: {TEXT: foo}
      - task: print
        vars: {TEXT: bar}
      - task: print
        vars: {TEXT: baz}

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

:::tip

The `output` option can also be specified by the `--output` or `-o` flags.

:::

## Interactive CLI application

When running interactive CLI applications inside Task they can sometimes behave
weirdly, especially when the [output mode](#output-syntax) is set to something
other than `interleaved` (the default), or when interactive apps are run in
parallel with other tasks.

The `interactive: true` tells Task this is an interactive application and Task
will try to optimize for it:

```yaml
version: '3'

tasks:
  default:
    cmds:
      - vim my-file.txt
    interactive: true
```

If you still have problems running an interactive app through Task, please open
an issue about it.

## Short task syntax

Starting on Task v3, you can now write tasks with a shorter syntax if they
have the default settings (e.g. no custom `env:`, `vars:`, `desc:`, `silent:` , etc):

```yaml
version: '3'

tasks:
  build: go build -v -o ./app{{exeExt}} .

  run:
    - task: build
    - ./app{{exeExt}} -h localhost -p 8080
```

## 任务监控 Watch tasks

With the flags `--watch` or `-w` task will watch for file changes
and run the task again. This requires the `sources` attribute to be given,
so task knows which files to watch.

The default watch interval is 5 seconds, but it's possible to change it by
either setting `interval: '500ms'` in the root of the Taskfile passing it
as an argument like `--interval=500ms`.

[gotemplate]: https://golang.org/pkg/text/template/
[minify]: https://github.com/tdewolff/minify/tree/master/cmd/minify
