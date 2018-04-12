---
title: Hexo的使用
tag: Hexo
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

### 一些学习心得
- hexo g --watch可以免去每次修改配置，主题配置，主题ejs和source文件夹中markdown文件后手动执行hexo g
- hexo s 默认是监视public的文件更改的，如果修改了配置或者source文件夹中markdown文件后，执行了hexo g，那么hexo本地服务会实时渲染新的更改。
- Hexo 使用 Nunjucks 来解析文章（旧版本使用 Swig，两者语法类似），内容若包含{% raw %} {{ }} {% endraw %}或{% raw %} {% %} {% endraw %}可能导致解析错误，您可以用 raw 标签包裹来避免潜在问题发生。在我一篇文章[《Django如何支持网站多语言切换》](/zh-CN/Django如何支持网站多语言切换？)中踩过坑。详情见[官方文档问答部分](https://hexo.io/docs/troubleshooting.html#Escape-Contents)。代码如下：
```python
{% raw %}
Hello {% trans %}
{% endraw %}
```
- 使用markdown后，引用的一些图片，网上大多使用图床解决，但是hexo支持相对路径引用图片，设置_config.yml中post_asset_folder: true，然后两种引用代码如下：
```
#首页不可以显示，需要点进文章页面才能显示的调用相对路径方式
![userhead](userhead.jpg)
#首页和文章页面都可以显示的调用相对路径图片方式
{% asset_img userhead.jpg userhead %}
```
![userhead](userhead.jpg)
{% asset_img userhead.jpg userhead %}
详情见[官方文档](https://hexo.io/docs/asset-folders.html#Tag-Plugins-For-Relative-Path-Referencing)。例如



