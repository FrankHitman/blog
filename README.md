# Frank's Technology Blog
基于hexo构建的blog代码仓库，用于渲染hithub pages的[个人博客](https://frankhitman.github.io "Frank's Blog")

## 项目说明
1. **博客框架**：
   - 这是一个使用 Hexo 框架搭建的静态博客
   - 当前使用的主题是 `yilia`
   - Hexo 版本是 4.0.0

2. **写新文章的方法**：
   - 文章应该放在 `source/_posts` 目录下
   - 文章格式为 Markdown（.md）
   - 文件名格式为 `:lang/:title.md`（根据配置）
   - 支持中文和英文文章（配置了多语言支持）

3. **部署到 GitHub Pages**：
   - 已经配置了 GitHub Pages 部署
   - 部署仓库是 `git@github.com:FrankHitman/frankhitman.github.io.git`
   - 部署分支是 `master`

要写新文章并部署，您可以按照以下步骤操作：

1. **创建新文章**：
```bash
hexo new post "文章标题"
```

2. **本地预览**：
```bash
hexo clean
hexo generate
hexo server
```

3. **部署到 GitHub Pages**：
```bash
hexo clean
hexo generate
hexo deploy
```

4. ** Help **:
```bash
MacBook-Pro:blog frank$ hexo -h
Usage: hexo <command>

Commands:
  clean     Remove generated files and cache.
  config    Get or set configurations.
  deploy    Deploy your website.
  generate  Generate static files.
  help      Get help on a command.
  init      Create a new Hexo folder.
  list      List the information of the site
  migrate   Migrate your site from other system to Hexo.
  new       Create a new post.
  publish   Moves a draft post from _drafts to _posts folder.
  render    Render files with renderer plugins.
  server    Start the server.
  version   Display version information.

Global Options:
  --config  Specify config file instead of using _config.yml
  --cwd     Specify the CWD
  --debug   Display all verbose messages in the terminal
  --draft   Display draft posts
  --safe    Disable all plugins and scripts
  --silent  Hide output on console

For more help, you can use 'hexo help [command]' for the detailed information
or you can check the docs: http://hexo.io/docs/

```


Ran tool
我看到您的博客已经按照语言分类了文章目录（zh-CN 和 en）。当您要写新文章时：

1. 中文文章放在 `source/_posts/zh-CN/` 目录下
2. 英文文章放在 `source/_posts/en/` 目录下

文章的基本格式应该是：
```markdown
---
title: 文章标题
date: 2024-03-21
categories: 
- 分类
tags:
- 标签1
- 标签2
---

文章内容...
```

## Hexo
Hexo command description and how to create new post which is specified language.

1. **基本命令**：
```bash
# 创建新文章
hexo new post "文章标题"

# 创建新页面
hexo new page "页面名称"

# 创建草稿
hexo new draft "草稿标题"

# 发布草稿
hexo publish "草稿标题"

# 生成静态文件
hexo generate 或 hexo g

# 启动本地服务器
hexo server 或 hexo s

# 清理缓存
hexo clean

# 部署网站
hexo deploy 或 hexo d
```

2. **指定文章存储目录的方法**：

目前您的博客配置中使用了 `:lang/:title.md` 的格式，这意味着：
- 中文文章会自动存储在 `source/_posts/zh-CN/` 目录
- 英文文章会自动存储在 `source/_posts/en/` 目录

如果您想要自定义文章存储位置，有以下几种方法：

a) **修改配置文件**：
在 `_config.yml` 中修改 `new_post_name` 参数：
```yaml
new_post_name: :category/:title.md  # 按分类存储
# 或
new_post_name: :year/:month/:title.md  # 按年月存储
```

b) **使用命令行参数**：
```bash
# 指定分类创建文章
hexo new post "文章标题" --path category/文章标题

# 指定完整路径创建文章
hexo new post "文章标题" --path custom/path/文章标题
```

3. **文章模板设置**：
您可以在 `scaffolds/post.md` 中设置新文章的默认模板。

模板：

Read file: scaffolds/post.md

4. **其他有用的命令**：
```bash
# 列出所有文章
hexo list post

# 列出所有草稿
hexo list draft

# 列出所有页面
hexo list page

# 生成 RSS
hexo generate-feed

# 生成站点地图
hexo generate-sitemap
```

5. **文章管理建议**：
- 建议使用有意义的文件名，避免使用中文或特殊字符
- 可以在文章开头添加更多元数据，如：
```yaml
---
title: 文章标题
date: 2024-03-21
updated: 2024-03-21
categories: 
  - 分类1
  - 分类2
tags:
  - 标签1
  - 标签2
description: 文章描述
keywords: 关键词1,关键词2
comments: true
---
```

6. **多语言支持**：
您的博客已经配置了多语言支持，可以通过以下方式创建不同语言的文章：
```bash
# 创建中文文章
hexo new post "中文标题" --lang zh-CN

# 创建英文文章
hexo new post "English Title" --lang en
```

## Hexo 图片显示问题
让我帮您分析这个问题。首先让我检查一下相关配置：


Ran tool
让我看看您的文章目录结构：


Ran tool
我看到您已经正确设置了 `post_asset_folder: true`，并且确实有文章对应的资源文件夹（比如 `hello-world/` 和 `SQLAlchemy-dirty-write-problem/`）。

在 Hexo 中，图片显示问题通常有以下几个原因：

1. **图片路径引用方式不正确**
在 Hexo 中，有两种正确的图片引用方式：

```markdown
# 方式1：使用相对路径（推荐）
![图片描述](图片名称.jpg)

# 方式2：使用 asset_img 标签
{% asset_img 图片名称.jpg 图片描述 %}
```

2. **图片文件夹结构**
正确的文件夹结构应该是：
```
source/_posts/
  ├── 文章名.md
  └── 文章名/
      └── 图片.jpg
```

3. **解决方案**：

a) 如果您使用的是相对路径，请确保：
- 图片文件确实存在于文章对应的资源文件夹中
- 图片路径引用正确

b) 如果您想使用 asset_img 标签，需要：
- 确保已安装 hexo-asset-image 插件
- 使用正确的语法

让我帮您检查一下是否安装了相关插件：


Read file: package.json
我注意到您的项目中缺少 `hexo-asset-image` 插件。建议您：

1. **安装 hexo-asset-image 插件**：
```bash
npm install hexo-asset-image --save
```

2. **使用正确的图片引用方式**：
```markdown
# 方式1：使用相对路径（推荐）
![图片描述](图片名称.jpg)

# 方式2：使用 asset_img 标签
{% asset_img 图片名称.jpg 图片描述 %}
```

3. **确保图片文件夹结构正确**：
- 每篇文章的图片应该放在与文章同名的文件夹中
- 例如：`source/_posts/zh-CN/文章名/图片.jpg`

4. **重新生成和部署**：
```bash
hexo clean
hexo generate
hexo deploy
```
### 首页不显示，详情页显示的问题
使用绝对路径解决。

这个问题通常是因为首页和文章详情页的路径不同导致的。让我帮您分析一下配置和解决方案。

首先检查一下您的主题配置：


Ran tool
我看到您使用的是 yilia 主题。这个问题主要有以下几种解决方案：

1. **使用绝对路径引用图片**
在 `_config.yml` 中设置 `relative_link: false`（您已经设置了这个选项）

2. **修改图片引用方式**
在文章中使用以下方式之一：

```markdown
# 方式1：使用 asset_img 标签（推荐）
{% asset_img 图片名称.jpg 图片描述 %}

# 方式2：使用绝对路径
![图片描述](/文章名/图片名称.jpg)

# 方式3：使用相对路径，但需要确保路径正确
![图片描述](./文章名/图片名称.jpg)
```

3. **修改主题模板**
如果您想继续使用相对路径，可以修改主题的模板文件。让我检查一下您的主题配置：


Read file: themes/yilia/_config.yml
基于以上分析，我建议您采用以下解决方案：

1. **最佳解决方案：使用 asset_img 标签**
```markdown
{% asset_img 图片名称.jpg 图片描述 %}
```

2. **如果不想修改现有文章，可以修改主题模板**
在 `themes/yilia/layout/_partial/article.ejs` 中添加图片路径处理逻辑。

3. **使用绝对路径**
在文章中使用绝对路径引用图片：
```markdown
![图片描述](/文章名/图片名称.jpg)
```

4. **修改 Hexo 配置**
在 `_config.yml` 中添加以下配置：
```yaml
post_asset_folder: true
relative_link: false
```

具体操作步骤：

1. **对于新文章**：
   - 使用 `hexo new post "文章名"` 创建新文章
   - 在同名文件夹中放入图片
   - 使用 asset_img 标签引用图片

2. **对于现有文章**：
   - 将图片移动到文章对应的资源文件夹中
   - 修改图片引用方式为 asset_img 标签或绝对路径

3. **重新生成和部署**：
```bash
hexo clean
hexo generate
hexo deploy
```

## deploy hexo problem
```
MacBook-Pro:blog frank$ hexo d
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
TypeError: mode must be int32 or null/undefined
    at copyFile (node:fs:3001:11)
    at promisified (/Users/frank/play/mygithub/blog/node_modules/bluebird/js/release/promisify.js:223:16)
    at /Users/frank/play/mygithub/blog/node_modules/hexo-fs/lib/fs.js:144:39
    at tryCatcher (/Users/frank/play/mygithub/blog/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._se
```

我看到您在部署到 GitHub Pages 时遇到了错误。这个错误提示是：`TypeError: mode must be int32 or null/undefined`，这通常与文件权限或 Node.js 版本有关。让我们一步步解决这个问题：

1. **首先检查 Node.js 版本**：


```bash
node -v
```
我看到您使用的是 Node.js v23.1.0，这是一个非常新的版本。这个错误可能是由于 Node.js 版本与 Hexo 的兼容性问题导致的。让我们尝试以下解决方案：

1. **清理部署缓存**：
```bash
rm -rf .deploy_git
```

2. **重新安装依赖**：
```bash
rm -rf node_modules
npm install
```

3. **使用较低版本的 Node.js**（推荐使用 Node.js 16.x 或 18.x LTS 版本）

您可以使用 nvm（Node Version Manager）来管理 Node.js 版本：

```bash
# 安装 nvm（如果还没有安装）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```
MacBook-Pro:blog frank$ brew install nvm
==> Downloading https://ghcr.io/v2/homebrew/core/nvm/manifests/0.40.1
################################################################################################################################# 100.0%
==> Fetching nvm
==> Downloading https://ghcr.io/v2/homebrew/core/nvm/blobs/sha256:7579c8f8fcf63b576c78e58006075ed8bf53049491cb63ed64f0bab2bf5f2942
################################################################################################################################# 100.0%
==> Pouring nvm--0.40.1.all.bottle.tar.gz
==> Caveats
Please note that upstream has asked us to make explicit managing
nvm via Homebrew is unsupported by them and you should check any
problems against the standard nvm install method prior to reporting.

You should create NVM's working directory if it doesn't exist:
  mkdir ~/.nvm

Add the following to your shell profile e.g. ~/.profile or ~/.zshrc:
  export NVM_DIR="$HOME/.nvm"
  [ -s "/usr/local/opt/nvm/nvm.sh" ] && \. "/usr/local/opt/nvm/nvm.sh"  # This loads nvm
  [ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/usr/local/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion

You can set $NVM_DIR to any location, but leaving it unchanged from
/usr/local/Cellar/nvm/0.40.1 will destroy any nvm-installed Node installations
upon upgrade/reinstall.

Type `nvm help` for further information.

Bash completion has been installed to:
  /usr/local/etc/bash_completion.d
==> Summary
🍺  /usr/local/Cellar/nvm/0.40.1: 10 files, 203.0KB
```

```
MacBook-Pro:blog frank$ vi ~/.bash_profile 
MacBook-Pro:blog frank$ 
MacBook-Pro:blog frank$ 
MacBook-Pro:blog frank$ source ~/.bash_profile 
MacBook-Pro:blog frank$ 
MacBook-Pro:blog frank$ nvm

Node Version Manager (v0.40.1)

Note: <version> refers to any version-like string nvm understands. This includes:
  - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
  - default (built-in) aliases: node, stable, unstable, iojs, system
  - custom aliases you define with `nvm alias foo`

 Any options that produce colorized output should respect the `--no-colors` option.

Usage:
  nvm --help                                  Show this message
    --no-colors                               Suppress colored output
  nvm --version                               Print out the installed version of nvm
  nvm install [<version>]                     Download and install a <version>. Uses .nvmrc if available and version is omitted.
   The following optional arguments, if provided, must appear directly after `nvm install`:
    -s                                        Skip binary download, install from source only.
    -b                                        Skip source download, install from binary only.
    --reinstall-packages-from=<version>       When installing, reinstall packages installed in <node|iojs|node version number>
    --lts                                     When installing, only select from LTS (long-term support) versions
    --lts=<LTS name>                          When installing, only select from versions for a specific LTS line
    --skip-default-packages                   When installing, skip the default-packages file if it exists
    --latest-npm                              After installing, attempt to upgrade to the latest working npm on the given node version
    --no-progress                             Disable the progress bar on any downloads
    --alias=<name>                            After installing, set the alias specified to the version specified. (same as: nvm alias <name> <version>)
    --default                                 After installing, set default alias to the version specified. (same as: nvm alias default <version>)
    --save                                    After installing, write the specified version to .nvmrc
  nvm uninstall <version>                     Uninstall a version
  nvm uninstall --lts                         Uninstall using automatic LTS (long-term support) alias `lts/*`, if available.
  nvm uninstall --lts=<LTS name>              Uninstall using automatic alias for provided LTS line, if available.
  nvm use [<version>]                         Modify PATH to use <version>. Uses .nvmrc if available and version is omitted.
   The following optional arguments, if provided, must appear directly after `nvm use`:
    --silent                                  Silences stdout/stderr output
    --lts                                     Uses automatic LTS (long-term support) alias `lts/*`, if available.
    --lts=<LTS name>                          Uses automatic alias for provided LTS line, if available.
    --save                                    Writes the specified version to .nvmrc.
  nvm exec [<version>] [<command>]            Run <command> on <version>. Uses .nvmrc if available and version is omitted.
   The following optional arguments, if provided, must appear directly after `nvm exec`:
    --silent                                  Silences stdout/stderr output
    --lts                                     Uses automatic LTS (long-term support) alias `lts/*`, if available.
    --lts=<LTS name>                          Uses automatic alias for provided LTS line, if available.
  nvm run [<version>] [<args>]                Run `node` on <version> with <args> as arguments. Uses .nvmrc if available and version is omitted.
   The following optional arguments, if provided, must appear directly after `nvm run`:
    --silent                                  Silences stdout/stderr output
    --lts                                     Uses automatic LTS (long-term support) alias `lts/*`, if available.
    --lts=<LTS name>                          Uses automatic alias for provided LTS line, if available.
  nvm current                                 Display currently activated version of Node
  nvm ls [<version>]                          List installed versions, matching a given <version> if provided
    --no-colors                               Suppress colored output
    --no-alias                                Suppress `nvm alias` output
  nvm ls-remote [<version>]                   List remote versions available for install, matching a given <version> if provided
    --lts                                     When listing, only show LTS (long-term support) versions
    --lts=<LTS name>                          When listing, only show versions for a specific LTS line
    --no-colors                               Suppress colored output
  nvm version <version>                       Resolve the given description to a single local version
  nvm version-remote <version>                Resolve the given description to a single remote version
    --lts                                     When listing, only select from LTS (long-term support) versions
    --lts=<LTS name>                          When listing, only select from versions for a specific LTS line
  nvm deactivate                              Undo effects of `nvm` on current shell
    --silent                                  Silences stdout/stderr output
  nvm alias [<pattern>]                       Show all aliases beginning with <pattern>
    --no-colors                               Suppress colored output
  nvm alias <name> <version>                  Set an alias named <name> pointing to <version>
  nvm unalias <name>                          Deletes the alias named <name>
  nvm install-latest-npm                      Attempt to upgrade to the latest working `npm` on the current node version
  nvm reinstall-packages <version>            Reinstall global `npm` packages contained in <version> to current version
  nvm unload                                  Unload `nvm` from shell
  nvm which [current | <version>]             Display path to installed node version. Uses .nvmrc if available and version is omitted.
    --silent                                  Silences stdout/stderr output when a version is omitted
  nvm cache dir                               Display path to the cache directory for nvm
  nvm cache clear                             Empty cache directory for nvm
  nvm set-colors [<color codes>]              Set five text colors using format "yMeBg". Available when supported.
                                               Initial colors are:
                                                  bygre
                                               Color codes:
                                                r/R = red / bold red
                                                g/G = green / bold green
                                                b/B = blue / bold blue
                                                c/C = cyan / bold cyan
                                                m/M = magenta / bold magenta
                                                y/Y = yellow / bold yellow
                                                k/K = black / bold black
                                                e/W = light grey / white
Example:
  nvm install 8.0.0                     Install a specific version number
  nvm use 8.0                           Use the latest available 8.0.x release
  nvm run 6.10.3 app.js                 Run app.js using node 6.10.3
  nvm exec 4.8.3 node app.js            Run `node app.js` with the PATH pointing to node 4.8.3
  nvm alias default 8.1.0               Set default node version on a shell
  nvm alias default node                Always default to the latest available node version on a shell

  nvm install node                      Install the latest available version
  nvm use node                          Use the latest version
  nvm install --lts                     Install the latest LTS version
  nvm use --lts                         Use the latest LTS version

  nvm set-colors cgYmW                  Set text colors to cyan, green, bold yellow, magenta, and white

Note:
  to remove, delete, or uninstall nvm - just remove the `$NVM_DIR` folder (usually `~/.nvm`)

```

# 安装并使用 Node.js 18 LTS
nvm install 18
nvm use 18

MacBook-Pro:blog frank$ node -v
v18.20.8
```

4. **修改部署配置**
让我检查一下您的部署配置：


Read file: _config.yml
建议按以下步骤解决：

1. **降级 Node.js 版本**（这是最关键的步骤）：
```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 重新打开终端后执行：
nvm install 18
nvm use 18
```

2. **清理并重新安装**：
```bash
# 清理缓存和生成的文件
rm -rf .deploy_git
rm -rf node_modules
rm -rf public

# 重新安装依赖
npm install

# 重新生成和部署
hexo clean
hexo generate
hexo deploy
```

3. **如果还是有问题，可以尝试修改部署配置**：
在 `_config.yml` 中修改部署配置：
```yaml
deploy:
  type: git
  repo: git@github.com:FrankHitman/frankhitman.github.io.git
  branch: master
  message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"
```

4. **确保 Git 配置正确**：
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

5. **检查 SSH 密钥**：
```bash
# 测试 SSH 连接
ssh -T git@github.com
```

如果您想继续使用当前的 Node.js 版本，可以尝试：

1. 更新 hexo-deployer-git：
```bash
npm uninstall hexo-deployer-git
npm install hexo-deployer-git --save
```

2. 或者尝试使用其他部署方式，比如：
```bash
npm install hexo-deployer-rsync --save
```

然后在 `_config.yml` 中修改部署配置：
```yaml
deploy:
  type: rsync
  host: <host>
  user: <user>
  root: <root>
  path: <path>
  delete: true
```

最终是这样解决的，升级依赖包：
```
npm uninstall hexo-deployer-git
npm install hexo-deployer-git@latest --save

npm audit fix
```

```
MacBook-Pro:blog frank$ npm install hexo-deployer-git@latest --save
npm warn deprecated fsevents@1.2.9: Upgrade to fsevents v2 to mitigate potential security issues

added 58 packages, removed 10 packages, changed 3 packages, and audited 494 packages in 14s

14 packages are looking for funding
  run `npm fund` for details

48 vulnerabilities (5 low, 9 moderate, 24 high, 10 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
MacBook-Pro:blog frank$ npm audit fix
```
