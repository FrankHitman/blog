---
title: go module的使用
lang: zh-CN
date: 2019-11-01 19:56:01
categories:
    - golang
tags: 
    - go module
    - vgo 
---

### 弃用dep，投入mod怀抱
说下原因，由于我在wallbox/pkg目录下放了其他工程可以使用的公共包，那么在我另起一个项目时候，
例如OM项目，需要引入pkg/auth，会报错如下：
```
Cannot use 'c' (type *"github.com/gin-gonic/gin".Context) as type *"siemens.com/wallbox/vendor/github.com/gin-gonic/gin"
```
<!--more-->
这是由于引入vendor导致的类型不匹配 ，go官方出品的mod完美解决这个问题

### mod特点
- mod不会在项目目录下创建vendor目录，它会把依赖下载到$GOPATH/pkg/mod目录中，
并且不同版本占用一个目录
- mod官方tool集成度高，只要生成另go.mod文件（通过 ``` go mod init siemens.com/wallbox ``` ），正常run的时候会自动检测依赖并下载
- 避免了上面vendor递归依赖导致的问题，它完全为不同项目新建不同的go.mod依赖关系

### mod使用
#### Go Module 版本规则
依赖规则由两个部分组成，前面一部分是包路径，后面一部分表示的是版本号。 
你会发现有两种版本号，一种是我们很熟悉的 git 标签，比如 v0.0.2，
另一种就比较复杂一些，它是：版本号 + 时间戳 +hash 
比如：v0.0.0-20190212224330-8d79a5489543，
它其实是精准的对应着一个 git log 记录，后面的哈希是去提交哈希的前 12 位。
```
commit 52fc5a8c37932b7e71405c53795a619f011993cc (HEAD -> master)
Author: frank <xxx@email.com>
Date:   Mon Aug 19 15:39:00 2019 +0800

    move some package from internal to pkg

```
则我的最新版本号应该为	```siemens.com/wallbox v0.0.0-20190819153900-52fc5a8c3793```

#### 引入本地依赖包
我在om项目目录下go.mod
```
siemens.com/wallbox v0.0.0-20190819153900-52fc5a8c3793
```

#### 使用 replace 将远程包替换为本地包服务
这时如果你执行 go build 的时候会报错，提示找不到 
```
go: siemens.com/wallbox/pkg@v0.0.0-20190819153900-52fc5a8c3793: unrecognized import path "siemens.com/wallbox/pkg" (parse https://siemens.com/wallbox/pkg?go-get=1: no go-import meta tags ())
go: error loading module requirements
```
是因为你没有把仓库推送到远程，所以无法下载。 
go module 提供了另外一个方案, 使用 replace, 编辑 go.mod 文件，在最后面添加：``` replace siemens.com/wallbox => ../wallbox ```
执行``` go mod tidy ```整理依赖包，该下载下载该删除删除
最终我的go.mod张这个样子：
```
module siemens.com/om

go 1.12

require (
	github.com/alecthomas/template v0.0.0-20190718012654-fb15b899a751
	github.com/gin-contrib/pprof v1.2.1
	github.com/gin-gonic/gin v1.4.0
	github.com/go-redis/redis v6.15.2+incompatible
	github.com/onsi/ginkgo v1.8.0 // indirect
	github.com/onsi/gomega v1.5.0 // indirect
	github.com/satori/go.uuid v1.2.0
	github.com/spf13/pflag v1.0.3
	github.com/spf13/viper v1.4.0
	github.com/swaggo/gin-swagger v1.2.0
	github.com/swaggo/swag v1.6.2
	github.com/willf/pad v0.0.0-20190207183901-eccfe5d84172
	siemens.com/wallbox v0.0.0-20190819153900-52fc5a8c3793
)

replace siemens.com/wallbox => ../wallbox
```

### 也遇到坑
#### 找不到go.mod
例如一开始使用精确到pkg的引入 ``` siemens.com/wallbox/pkg v0.0.0-20190819153900-52fc5a8c3793 ``` 会报在被导入的目录下找不到go.mod（但是此文件在wallbox目录下）
```
go: parsing ../wallbox/pkg/go.mod: open /Users/frank/work/siemens.com/wallbox/pkg/go.mod: no such file or directory
go: error loading module requirements
```
#### 本地包的地址
另外远程包替换为本地包服务时候，本地包的地址要么是绝对地址要么是相对于本项目的相对地址，设置$GOPATH/src/xxxx是无效的。

#### 版本不一致
```
go: github.com/robfig/cron@v0.0.0-20190711170238-45fbe1491cdd: go.mod has post-v0 module path "github.com/robfig/cron/v3" at revision 45fbe1491cdd
go: error loading module requirements
```
删除$GOPATH/pkg/mod/github.com/robfig/ 相应的已经下载版本，删除go.mod目录中相应的依赖声明 ``` github.com/robfig/cron v0.0.0-20190711170238-45fbe1491cdd ```

#### 依赖的包版本太低，报错
执行go get会自动下载到$GOPATH/pkg目录中新的版本。
```
go get -u github.com/swaggo/gin-swagger
```

#### 升级大版本
升级大版本时候的包引入需要在import时候加入v2，例如从1.x.x -> 2.0.0的时候，go.mod，与包import如下更改
```
module siemens.com/om
go 1.12
require (
	github.com/casbin/casbin/v2 v2.1.0
	github.com/casbin/gorm-adapter/v2 v2.0.3
)
```

```go
import (
	"github.com/casbin/casbin/v2"
	"github.com/casbin/casbin/v2/model"
	"github.com/casbin/gorm-adapter/v2"
)
```
#### 清理掉不需要的依赖项
通过 go mod tidy 对版本进行更新，它会自动清理掉不需要的依赖项，同时可以将依赖项更新到当前版本。

### 感叹dep
没能成为官方的包依赖工具，为此社区成员与Google官方团队还吵了一架，=_=。替社区心疼2s。
不过以后肯定是go module的天下，我注意到GitHub上面的第三方库都开始往go module转了。

reference: 
- [如何使用modules](https://juejin.im/post/5c8e503a6fb9a070d878184a)
- [Go Module 引入本地自定义包]( http://www.r9it.com/20190611/go-mod-use-dev-package.html)
- [Go Modules详解](https://objcoding.com/2018/09/13/go-modules/)