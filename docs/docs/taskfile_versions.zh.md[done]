---
slug: /taskfile-versions/
sidebar_position: 9
---

# Taskfile 版本

Taskfile 的语法和功能随时间演变。这篇文档描述了每个版本的变化，以及如何升级旧的 Taskfile。

## Taskfile 的版本意味着什么

Taskfile 的版本是绑定 Task 的版本的。例如 Taskfile 的版本是 `2`，表示 Task 应该使用 `v2.0.0` 版本来执行。

Taskfile 文件的 `version:` 关键字接受语义化字符串，所以 `2`, `2.0` 或 `2.0.0` 都可以。
如果使用版本号 `2.0`，那么 Task 就不会使用 `2.1` 的功能，但如果使用版本号 `2`, 那么任意 `2.x.x` 版本中的功能都是可用的，但 `3.0.0+` 的版本不可用。

## Version 1

> 注意：Task v3.0.0 以后就不再支持 Taskfiles 的 1 版本了。

最早的 `Taskfile` 并不支持 `version:` 关键字，因为 YAML 文档中的顶层字段都是任务，例如：

```yaml
echo:
  cmds:
    - echo "Hello, World!"
```

变量的优先级也不同：

1. 调用变量
2. 环境变量
3. 任务内变量
4. `Taskvars.yml` 中定义变量

## Version 2.0

到了 2.0 版本，我们引入了 `version:` 关键字，在不破坏已存在的 Taskfiles 的前提下，在 Task 中引入新功能。
新语法如下：

```yaml
version: '2'

tasks:
  echo:
    cmds:
      - echo "Hello, World!"
```

2.0 版本允许你在 Taskfile 中直接设置全局变量，而不需要创建 `Taskvars.yml` 文件:

```yaml
version: '2'

vars:
  GREETING: Hello, World!

tasks:
  greet:
    cmds:
      - echo "{{.GREETING}}"
```

变量的优先级调整为：

1. 任务内变量
2. 调用变量
3. Taskfile 中变量
4. Taskvars 文件中定义变量
5. 环境变量

新增全局配置项，允许配置变量扩展范围（默认为 2）:

```yaml
version: '2'

expansions: 3

vars:
  FOO: foo
  BAR: bar
  BAZ: baz
  FOOBAR: "{{.FOO}}{{.BAR}}"
  FOOBARBAZ: "{{.FOOBAR}}{{.BAZ}}"

tasks:
  default:
    cmds:
      - echo "{{.FOOBARBAZ}}"
```

## 2.1 版本

2.1 版本包含了 `output` 选项，控制命令如何输出到控制台（查看 [documentation][output] 获得更多消息）:

```yaml
version: '2'

output: prefixed

tasks:
  server:
    cmds:
      - go run main.go
  prefix: server
```

这个版本开始允许忽略命令或任务的错误消息（[查看文档][ignore_errors]）:

```yaml
version: '2'

tasks:
  example-1:
    cmds:
      - cmd: exit 1
        ignore_error: true
      - echo "This will be print"

  example-2:
    cmds:
      - exit 1
      - echo "This will be print"
    ignore_error: true
```

## Version 2.2

2.2 版本在全局增加了 `includes` 选项，来引入其它 Taskfiles:

```yaml
version: '2'

includes:
  docs: ./documentation # will look for ./documentation/Taskfile.yml
  docker: ./DockerTasks.yml
```

## Version 2.6

2.6 版本增加任务的先决条件字段 `preconditions`。

```yaml
version: '2'

tasks:
  upload_environment:
    preconditions:
      - test -f .env
    cmds:
      - aws s3 cp .env s3://myenvironment
```

查看文档 [documentation][includes]

[output]: usage.md#output-syntax
[ignore_errors]: usage.md#ignore-errors
[includes]: usage.md#including-other-taskfiles

## Version 3

`v3` 版本的一些主要变化有:

- Task 的日志使用彩色输出
- 支持类 `.env` 文件
- 新增 `label:` 设置后会覆盖输出日志中的任务名字
- 新增 `method:` 设置全局默认方法，未设置时 Task 的默认方法变更为 `checksum`
- `status:` 中新增两个魔法变量: `CHECKSUM` 和 `TIMESTAMP`，用来显示 `sources:` 中文件的 md5 值以及最后变更时间
- 同时，`TASK` 变量始终表示当前任务的名字
- CLI 中的变量总是被当做全局变量
- `includes` 中新增 `dir:`，允许选择引入文件的具体目录:

```yaml
includes:
  docs:
    taskfile: ./docs
    dir: ./docs
```

- 语法精简。下面所有的语法是相同的:

```yaml
version: '3'

tasks:
  print:
    cmds:
      - echo "Hello, World!"
```

```yaml
version: '3'

tasks:
  print:
    - echo "Hello, World!"
```

```yaml
version: '3'

tasks:
  print: echo "Hello, World!"
```

- 一个主要的重构是在变量处理上。现在的方式更容易理解。 `expansions:` 因为不必要而被移除。下面是 Task 处理变量的顺序，每一层都可以使用上一层的变量，也可以覆盖变量。
	- 环境变量
	- 全局或 CLI 变量
	- 调用变量
	- Task 内的变量
