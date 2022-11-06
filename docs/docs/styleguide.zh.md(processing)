---
slug: /styleguide/
sidebar_position: 5
---

# 风格指南

这是对 `Taskfile.yml` 文件的官方风格指南。包含了一些基本的介绍帮助 Taskfile 简洁易懂。

本文包含的风格指南是建议性质，并不一定要强制遵循。可以随意提出不同观点，如果有目标或想法。
当然，也可以提交 pull request 帮助完善风格指南。

## 名字用 `Taskfile.yml` 不要用 `taskfile.yml`

```yaml
# bad
taskfile.yml


# good
Taskfile.yml
```

这一点对 Linux 用户很重要。Windows 和 macOS 是大小写不敏感的文件吸血，所以叫 `taskfile.yml` 也可以正常使用，
虽然官方使用的不是这个名字。在 Linux 中，只有 `Taskfile.yml` 才能正常使用。

## 关键字的正确顺序

- `version:`
- `includes:`
- 一个可选配置命令，比如 `output:`, `silent:`, `method:` and `run:`
- `vars:`
- `env:`, `dotenv:`
- `tasks:`

## 缩进用 2 个空格

这是最常用的 YAML 风格，Task 也这样建议。

```yaml
# bad
tasks:
    foo:
        cmds:
            - echo 'foo'


# good
tasks:
  foo:
    cmds:
      - echo 'foo'
```

## 用空行分隔不同模块

```yaml
# bad
version: '3'
includes:
  docker: ./docker/Taskfile.yml
output: prefixed
vars:
  FOO: bar
env:
  BAR: baz
tasks:
  # ...


# good
version: '3'

includes:
  docker: ./docker/Taskfile.yml

output: prefixed

vars:
  FOO: bar

env:
  BAR: baz

tasks:
  # ...
```

## 用空行分隔不同任务

```yaml
# bad
version: '3'

tasks:
  foo:
    cmds:
      - echo 'foo'
  bar:
    cmds:
      - echo 'bar'
  baz:
    cmds:
      - echo 'baz'


# good
version: '3'

tasks:
  foo:
    cmds:
      - echo 'foo'

  bar:
    cmds:
      - echo 'bar'

  baz:
    cmds:
      - echo 'baz'
```

## 用大写字母命名变量

```yaml
# bad
version: '3'

vars:
  binary_name: myapp

tasks:
  build:
    cmds:
      - go build -o {{.binary_name}} .


# good
version: '3'

vars:
  BINARY_NAME: myapp

tasks:
  build:
    cmds:
      - go build -o {{.BINARY_NAME}} .
```

## 使用变量名字时括号内不加空格

```yaml
# bad
version: '3'

tasks:
  greet:
    cmds:
      - echo '{{ .MESSAGE }}'


# good
version: '3'

tasks:
  greet:
    cmds:
      - echo '{{.MESSAGE}}'
```

这种习惯是大多数人在 Go 模板中使用的。

## 用横线分隔任务名字中的单词

```yaml
# bad
version: '3'

tasks:
  do_something_fancy:
    cmds:
      - echo 'Do something'


# good
version: '3'

tasks:
  do-something-fancy:
    cmds:
      - echo 'Do something'
```

## Use colon for task namespacing

```yaml
# good
version: '3'

tasks:
  docker:build:
    cmds:
      - docker ...

  docker:run:
    cmds:
      - docker-compose ...
```

This is also done automatically when using included Taskfiles.
