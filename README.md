# Go Mod 官方中文文档

<div align=center><img width="150" height="150" src="docs/img/Go.png"/></div>

* 🚀本仓库是 `Go Mod` 的相关官方文档翻译。
* 👏欢迎任何人加入翻译的工作。
* 🌟你的star就是我的动力。

Go 自从 1.11 版本以来就包含了对版本化模块 (versioned modules) 的[支持](https://golang.org/design/24301-versioned-go)。最初的原型`vgo`是在2018年2月发布的，2018年7月，版本化模块正式登陆Go的主分支。

在 `Go 1.14` 中，Go Module已经可以投入生产环境中使用，并且鼓励所有用户从其他的依赖管理工具迁移过来。如果在迁移过程中使用Go工具链出现了问题，可以去看[issues](https://github.com/golang/go/wiki/Modules#github-issues)。

## 更新日志

### Go 1.14

详细请见：[Go 1.14 发布日志](https://golang.org/doc/go1.14#go-command)

- 当主模块包含一个顶层的vendor，且`go.mod`中指定了Go的版本在1.14或以上的时候，go命令行会默认带上`-mod=vendor`的参数；
- 当`go.mod`文件是只读且没有顶层vendor目录的时候，`-mod=readonly`将被设为默认值；
- 新参数`-modcacherw`，模块缓存中的文件权限也会被复制到go命令创建的新目录（之前是全部只读）；
- 当`GO111MODULE=on`时，如果没有`go.mod`文件，大多数go命令行功能会被限制；
- Go命令行现在支持仓库的子版本了；

### Go 1.13

详情请见：[Go 1.13 发布日志](https://golang.org/doc/go1.13#modules)

- Go命令行现在默认从公共Go模块镜像 https://proxy.golang.org 下载，而且默认会从 https://sum.golang.org 校验checksum的值；
    - 如果你是私有的代码，那就应该配置`GOPRIVATE`参数（比如说`go env -w GOPRIVATE=*.corp.com,github.com/secret/repo`），或者更详细一点，使用`GONOPROXY`或者`GONOSUMDB`，这种方式比较少见，详见[文档](https://golang.org/cmd/go/#hdr-Module_configuration_for_non_public_modules)
- 当`GO111MODULE=auto`时，就算`go.mod`位于`GOPATH`内也能正确激活“模块模式（module-mode）”了（在之前的版本中，GOPATH内无法激活模块模式）；
- `go get`的参数发生了变化：
    - `go get -u`现在只升级当前package下的直接依赖和间接依赖，不再检查整个模块了；
    - 从模块的根目录执行`go get -u ./...`现在会升级所有你的模块的直接和间接依赖，并且现在会排除测试依赖；
    - `go get -u -t ./...`类似，不过也会升级测试依赖；
    - `go get`现在不再支持`-m`参数了（因为一些改动导致它和`-d`参数的功能几乎重合了，所以你现在可以使用`go get -d foo`来代替`go get -m foo`）。

## 目录

### 介绍 ([Introduction](https://blog.golang.org/using-go-modules))

该系列文章官方发布于 `2019年3月19日` 。

该部分已由[GCTT](https://github.com/studygolang/GCTT) ([studygolang](https://studygolang.com/)) 翻译，下面的链接可以直达。

- [x] 第一部分 — 开始使用 Go Modules | [翻译](https://studygolang.com/articles/19334) / [原文](https://blog.golang.org/using-go-modules)
- [x] 第二部分 — 迁移到 Go Modules | [翻译](https://studygolang.com/articles/17780) / [原文](https://blog.golang.org/migrating-to-go-modules)
- [x] 第三部分 — 发布 Go Modules | [翻译](https://studygolang.com/articles/25129) / [原文](https://blog.golang.org/publishing-go-modules)
- [x] 第四部分 — Go Modules: v2 和未来 | [翻译](https://studygolang.com/articles/25130) / [原文](https://blog.golang.org/v2-go-modules)

### 文档 ([GitHub Wiki](https://github.com/golang/go/wiki/Modules))

<details>
<summary><strong>快速上手</strong></summary>
<div>

## 快速上手

#### 示例

详细的内容在本页面的其他部分都会介绍，现在让我们先来从一个简单的例子入手，看看如何快速创建一个模块。

首先在`GOPATH`以外的地方创建一个目录，并初始化代码仓库（可选）：

```
$ mkdir -p /tmp/scratchpad/repo
$ cd /tmp/scratchpad/repo
$ git init -q
$ git remote add origin https://github.com/my/repo
```

初始化一个新的模块：

```
$ go mod init github.com/my/repo

go: creating new go.mod: module github.com/my/repo
```

写下你的代码：

```
$ cat <<EOF > hello.go
package main

import (
    "fmt"
    "rsc.io/quote"
)

func main() {
    fmt.Println(quote.Hello())
}
EOF
```

构建并运行：

```
$ go build -o hello
$ ./hello

Hello, world.
```

`go.mod`文件已经被自动更新了，包含了你的依赖包和版本号，版本号标签严格遵循[semver](https://semver.org/)规范。

```
$ cat go.mod

module github.com/my/repo

require rsc.io/quote v1.5.2
```

#### 日常使用流程

可以发现在上面的例子中，我们都没有使用到`go get`命令。

你的日常使用流程可以是：

- 在你的`.go`代码中根据需要添加`import`声明；
- 一些标准的命令例如`go build`或`go test`会自动添加新的依赖包以此来满足`imports`的需要（自动更新`go.mod`文件并且下载新的依赖包）；
- 如果需要的话，可以在`go get`命令或在编辑`go.mod`文件的时候指明版本号，例如`go get foo@v1.2.3`，`go get foo@master` (`foo@default`)，`go get foo@e3702bed2`。

其他你可能也经常会用到的功能也简单介绍一下：

- `go list -m all` —— 列出所有在build过程中会被直接和间接依赖的包的版本号；（[详情](https://github.com/golang/go/wiki/Modules#version-selection)）
- `go list -u -m all` —— 列出所有依赖包可用的升级；（[详情](https://github.com/golang/go/wiki/Modules#how-to-upgrade-and-downgrade-dependencies)）
- `go get -u ./...`或`go get -u=patch ./...`（在模块的根目录执行）—— 升级所有依赖包到最新版本（无视预发布的版本）；（[详情](https://github.com/golang/go/wiki/Modules#how-to-upgrade-and-downgrade-dependencies)）
- `go mod tidy` —— 从`go.mod`中清理所有不再需要的依赖包，并且增加其他需要的依赖包；（[详情](https://github.com/golang/go/wiki/Modules#how-to-prepare-for-a-release)）
- 在`go.mod`中使用`replace`可以使用其他依赖路径，比如fork的仓库、本地的拷贝或精确的版本号（例如`replace example.com/project/foo => ../foo`）；（[详情](https://github.com/golang/go/wiki/Modules#when-should-i-use-the-replace-directive)）
- `go mod vendor`一个可选的步骤，用于创建一个`vendor`路径。（[详情](https://github.com/golang/go/wiki/Modules#how-do-i-use-vendoring-with-modules-is-vendoring-going-away)）

在你读完接下来的“新概念”中的四小节内容后，足够应付大部分项目了。重新浏览目录也能帮助你熟悉Go Modules的一些更细节的讨论。

</div>
</details>

<details>
<summary><strong>新的概念</strong></summary>
<div>

## 新的概念

接下来的几个小节将会详细介绍Go Modules的几个概念。更详细的细节请参考40分钟的介绍：[GopherCon Sg 2018](https://www.youtube.com/watch?v=F8nrpe0XWRg&list=PLq2Nv-Sh8EbbIjQgDzapOFeVfv5bGOoPE&index=3&t=0s)，[官方文档](https://golang.org/design/24301-versioned-go)或[最初的vgo系列](https://research.swtch.com/vgo)

#### 模块

一个模块（module）定义为一系列相关的Go包（package）的集合，这些包作为一个单一单元被版本化。（译注：Go package其实就是一个目录下的单个或多个`.go`文件组成的包。）

模块记录了详细的依赖关系，且一个模块的构建过程也是可复现的。

大多数情况下，一个有版本控制的代码仓库，其根目录下只会包含一个模块。（单仓库多模块也是支持的，但是相较于单仓库单模块，会导致更多的版本管理的工作）

总结一下版本库（repo）、模块（module）和包（package）之间但关系：

- 一个版本库包含一个或多个Go模块；
- 每个模块包含一个或多个Go包；
- 每个包由一个目录下的单个或多个`.go`文件组成。

模块必须严格按照[semver](https://semver.org/)规范来打上有意义的版本号，通常情况下遵循格式`v(major).(minor).(patch)`，比如`v0.1.0`，`v1.2.3`，或者`v1.5.0-rc.1`。开头的`v`是必需的。如果使用Git，请在发布的时候用[tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging)标上版本号。公有库、私有库和镜像仓库都受到支持（见[下方](https://github.com/golang/go/wiki/Modules#are-there-always-on-module-repositories-and-enterprise-proxies)FAQ）

#### go.mod

#### 版本选择

#### 有意义的版本号

</div>
</details>

- [ ] 如何使用 Modules
- [ ] 迁移到 Modules
- [ ] 其他参考资料
- [ ] 从最初的 Vgo 发布后做的改变
- [ ] GitHub Issues
- [ ] FAQs

## 贡献

提供相关翻译至`docs/`目录下即可，待校对完毕后，README由我来更新就好。
