# API 参考

## 命令行 CLI

Task 命令行工具的语法如下:

```bash
task [--flags] [tasks...] [-- CLI_ARGS...]
```

> NOTE:
> 如果使用了 `--`，剩下的参数都会被传给内置的 `CLI_ARGS` 变量

| 缩写 | 标记 | 类型 | 默认 | 说明 |
| - | - | - | - | - |
| `-c` | `--color` | `bool` | `true` | 彩色输出，默认开启。设置为 false 或使用 `NO_COLOR=1` 禁用 |
| `-C` | `--concurrency` | `int` | `0` | 限制并行任务的数量，0 表示不限制 |
| `-d` | `--dir` | `string` | 项目工作目录 | 设置执行目录 |
| `-n` | `--dry` | `bool` | `false` | 编译并按顺序打印任务，但不真的执行 |
| `-x` | `--exit-code` | `bool` | `false` |  透传 task 命令的结束状态码 |
| `-f` | `--force` | `bool` | `false` | 强制执行，即使任务已经更新到最新状态 |
| `-h` | `--help` | `bool` | `false` | 显示 Task 的帮助信息 |
| `-i` | `--init` | `bool` | `false` | 在当前目录创建新的 Taskfile.yaml 文件 |
| `-I` | `--interval` | `string` | `5s` | 设置 `--watch` 模式下的监控时间，默认 5 秒，参数必须符合 [Go Duration](https://pkg.go.dev/time#ParseDuration) |
| `-l` | `--list` | `bool` | `false` | 列出当前文件的全部任务及对应描述 |
| `-a` | `--list-all` | `bool` | `false` | 列出全部任务，可选是否显示描述 |
| `-o` | `--output` | `string` | Taskfile 中设置的值 或 `intervealed` | 设置输出风格: [`interleaved`/`group`/`prefixed`]. |
|      | `--output-group-begin` | `string` | | 输出到任务组之前的消息模板 |
|      | `--output-group-end` | `string` | | 输出到任务组之后的消息模板 |
| `-p` | `--parallel` | `bool` | `false` | 并发执行任务 |
| `-s` | `--silent` | `bool` | `false` | 禁止打印命令 |
|      | `--status` | `bool` | `false` | 如果有任务未更新，会返回非 0 状态码 |
|      | `--summary` | `bool` | `false` | 显示任务摘要信息 |
| `-t` | `--taskfile` | `string` | `Taskfile.yml` 或 `Taskfile.yaml` | |
| `-v` | `--verbose` | `bool` | `false` | 输出详情 |
|      | `--version` | `bool` | `false` | 显示版本号 |
| `-w` | `--watch` | `bool` | `false` |  开启任务监控 |

## 特殊变量

系统模板中可以使用一些特殊变量:

| 变量名 | 描述 |
| - | - |
| `CLI_ARGS` | 外部输入参数，在命令行中 `--` 之后的所有内容 |
| `TASK` | 当前任务名称 |
| `ROOT_DIR` | 根 Taskfile 目录的绝对路径 |
| `TASKFILE_DIR` | 引用 Taskfile 目录的绝对路径 |
| `CHECKSUM` | `sources` 中配置的文件列表的 checksum 值，只能 `status` 配合使用，并且检查方法设置为 `checksum` |
| `TIMESTAMP` | `sources` 中配置的文件列表中最大的文件时间戳，只能 `status` 配合使用，并且检查方法设置为 `timestamp` |

## 环境变量

设置某些环境变量可以调整 Task 的默认行为。

| 变量 | 默认 | 描述 |
| - | - | - |
| `TASK_TEMP_DIR` | `.task` | 临时目录，可以是相对路径也可以是绝对路径 |
| `TASK_COLOR_RESET` | `0` | 白色色值 |
| `TASK_COLOR_BLUE` | `34` | 蓝色色值 |
| `TASK_COLOR_GREEN` | `32` | 绿色色值 |
| `TASK_COLOR_CYAN` | `36` | 青色色值 |
| `TASK_COLOR_YELLOW` | `33` | 黄色色值 |
| `TASK_COLOR_MAGENTA` | `35` | 洋红色色值 |
| `TASK_COLOR_RED` | `31` | 红色色值 |

## Schema

### Taskfile

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| `version` | `string` | | Taskfile 版本号，当前版本是 `3` |
| `output` | `string` | `interleaved` | 输出模式，支持: `interleaved`, `group` 和 `prefixed` |
| `method` | `string` | `checksum` | Taskfile 默认文件检查方法，可以覆盖的任务基础属性，支持：`checksum`, `timestamp` 和 `none` |
| `includes` | [`map[string]Include`](#include) | | 额外引入的 Taskfile |
| `vars` | [`map[string]Variable`](#variable) | | 一组全局变量 |
| `env` | [`map[string]Variable`](#variable) | | 一组全局环境变量 |
| `tasks` | [`map[string]Task`](#task) | | 一组任务定义 |
| `silent` | `bool` | `false` | 全部任务的默认 'silent' 选项，如果是 `false`, 那么可以在任务自己的基础属性中进行覆盖设置 |
| `dotenv` | `[]string` | | 将被解析的 `.env` 文件列表 |
| `run` | `string` | `always` | Taskfile 中默认的 `run` 设置。支持: `always`, `once` 和 `when_changed` |
| `interval` | `string` | `5s` | 使用 `--watch` 时设置不同的间隔时间, 默认 5 秒。这个字符串需要符合 [Go Duration](https://pkg.go.dev/time#ParseDuration). |

### Include

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| `taskfile` | `string` | | 被引用文件的路径或目录。如果是目录，Task 会查找叫 `Taskfile.yml` 或 `Taskfile.yaml` 文件。如果是相对路径，就会解析相对路径中的 Taskfile |
| `dir` | `string` | The parent Taskfile directory | 被引用文件执行时的工作目录 |
| `optional` | `bool` | `false` | 设置为 `true` 时, 文件不存在也不会报错 |
| `internal` | `bool` | `false` | 禁止通过命令行调用被引用文件中的任务。这些任务在使用 `--list` 时也会显示出来 |
| `aliases` | `[]string` | | 设置被引用文件的命名空间别名 |
| `vars` | `map[string]Variable` | | 一组给被引用文件设置的变量 |

> NOTE:
> 直接赋值一个字符串，与将字符串赋值给 `taskfile` 属性的作用是相同的。
>
> ```yaml
> includes:
>   foo: ./path
> ```

### Task

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| `cmds` | [`[]Command`](#command) | | 可以执行的命令列表 |
| `deps` | [`[]Dependency`](#dependency) | | 依赖任务，在当前任务执行前会并行执行这些依赖任务 |
| `label` | `string` | | 覆盖输出的任务名字，可以使用变量 |
| `desc` | `string` | | 任务简述，当使用 `task --list` 时显示 |
| `summary` | `string` | | 任务详述，调用 `task --summary [task]` 时显示 |
| `aliases` | `[]string` | | 可以调用任务的别名列表 |
| `sources` | `[]string` | | 执行任务前的资源检查列表，与 `checksum` 和 `timestamp` 方法相关，可以是文件路径或星号 |
| `generates` | `[]string` | | 通过任务生成的文件列表。与 `timestamp` 方法相关。可以是文件路径或星号 |
| `status` | `[]string` | | 检查是否应当执行当前任务的命令列表，结果可能导致跳过当前任务。这个方法会覆盖 `method`, `sources` 和 `generates` |
| `preconditions` | [`[]Precondition`](#precondition) | | 检查当前任务的依赖条件列表，如果条件不满足，任务报错 |
| `dir` | `string` | | 任务的执行目录，默认是当前工作目录 |
| `vars` | [`map[string]Variable`](#variable) | | 一组可以在任务内使用的变量 |
| `env` | [`map[string]Variable`](#variable) | | 一组可以在任务的命令中适用的环境变量 |
| `silent` | `bool` | `false` | 隐藏任务名字和执行日志，执行命令的输出依然会输出到 `STDOUT` 和 `STDERR`。与 `--list` 标记一同使用时，会隐藏任务的详述 |
| `interactive` | `bool` | `false` | 标记 task 中的命令是有交互 |
| `internal` | `bool` | `false` | 禁止命令行调用当前任务，但会显示在 `--list` 的输出列表中 |
| `method` | `string` | `checksum` | 定义如何检查任务是否更新，`timestamp` 会对比源文件和生成文件的时间戳，`checksum` 会检查 checksum (你可能需要在 .gitignore 中忽略 .task 目录)。 `none` 会跳过检查直接执行 task。 |
| `prefix` | `string` | | 定义 task 的输出前缀。只在输出模式为 `prefixed` 的时候生效 |
| `ignore_error` | `bool` | `false` | 执行命令的时候忽略错误，继续执行 |
| `run` | `string` | Taskfile 中全局声明的值或 `always` | 定义 task 是否应该被执行多次，支持：`always`, `once` 和 `when_changed` |

> NOTE:
> 以下语法也有效，定义内容会被设置给 `cmds` ，其它属性会使用默认值:
> 
> ```yaml
> tasks:
>   foo: echo "foo"
> 
>   foobar:
>     - echo "foo"
>     - echo "bar"
> 
>   baz:
>     cmd: echo "baz"
> ```
> 

### Dependency

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| `task` | `string` | | 作为依赖，需要执行的 task |
| `vars` | [`map[string]Variable`](#variable) | | 可选变量，会传递给 task |

> NOTE:
> 如果不需要设置变量，可以只声明一个数组（会被赋值给 `task`）:
> 
> ```yaml
> tasks:
>   foo:
>     deps: [foo, bar]
> ```
> 

### Command

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| `cmd` | `string` | | 要执行的 shell 命令 |
| `silent` | `bool` | `false` | 忽略部分日志输出，注意，日志依然会向 STDOUT 和 SDTERR 输出 |
| `task` | `string` | | 执行另一个 task，而不执行命令。不能与 `cmd` 同时设置 |
| `vars` | [`map[string]Variable`](#variable) | | 调用其它 task 时，可以传递变量，只能与 `task` 一起使用 |
| `ignore_error` | `bool` | `false` | 执行命令的时候忽略错误，继续执行 |
| `defer` | `string` | | 同 `cmd`, 但不立即执行，而延迟到 task 的最后执行。不能与 `cmd` 一同使用 |

> NOTE:
> 如果配置了一个字符串，则值传递给 `cmd`:
> 
> ```yaml
> tasks:
>   foo:
>     cmds:
>       - echo "foo"
>       - echo "bar"
> ```
> 

### Variable

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| *itself* | `string` | | 变量的静态值 |
| `sh` | `string` | | shell 命令，输出 (`STDOUT`) 值传递给变量 |

> NOTE:
> 静态和动态变量语法不同，如下：
> 
> ```yaml
> vars:
>   STATIC: static
>   DYNAMIC:
>     sh: echo "dynamic"
> ```
> 

### Precondition

| 属性 | 类型 | 默认 | 描述 |
| - | - | - | - |
| `sh` | `string` | | 要执行的命令，如果返回非 0 状态码，task 停止执行并返回错误 |
| `msg` | `string` | | 可选提示，条件不满足时显示 |

> NOTE:
> 不设置错误消息时，可以直接设置 precondition 值，省略 `sh`:
> 
> ```yaml
> tasks:
>   foo:
>     precondition: test -f Taskfile.yml
> ```
> 
