---
title: Hexo的使用
lang: zh-CN
date: 2018-05-28 16:45:01
categories:
    - nodejs
tags: 
    - Hexo 
---

## 一些学习心得
- hexo g --watch可以免去每次修改配置，主题配置，主题ejs和source文件夹中markdown文件后手动执行hexo g
- hexo s 默认是监视public的文件更改的，如果修改了配置或者source文件夹中markdown文件后，执行了hexo g，那么hexo本地服务会实时渲染新的更改。
<!--more-->
- Hexo 使用 Nunjucks 来解析文章（旧版本使用 Swig，两者语法类似），内容若包含{% raw %} {{ }} {% endraw %}或{% raw %} {% %} {% endraw %}可能导致解析错误，您可以用 raw 标签包裹来避免潜在问题发生。在我一篇文章[《Django如何支持网站多语言切换》](/zh-CN/Django如何支持网站多语言切换？)中踩过坑。详情见[官方文档问答部分](https://hexo.io/docs/troubleshooting.html#Escape-Contents)。代码如下：
```python
{% raw %}
Hello {% trans %}
{% endraw %}
```
- 使用markdown后，引用的一些图片，网上大多使用图床解决，但是hexo支持相对路径引用图片，[官方文档](https://hexo.io/docs/asset-folders.html#Tag-Plugins-For-Relative-Path-Referencing)，设置_config.yml中`post_asset_folder: true`，然后两种引用代码如下：
```
#首页不可以显示，需要点进文章页面才能显示的调用相对路径方式
![userhead](userhead.jpg)
#首页和文章页面都可以显示的调用相对路径图片方式
{% asset_img userhead.jpg userhead %}
```
![userhead](userhead.jpg)
{% asset_img userhead.jpg userhead %}
- 首页显示文章太长，需要截断，使用`<!--more-->`
- 如何使Hexo支持流程图和时序图？
需要安装两个插件 [hexo-filter-flowchart](https://github.com/bubkoo/hexo-filter-flowchart)和[hexo-filter-sequence](https://github.com/bubkoo/hexo-filter-sequence)；并且这两插件都依赖[deep-assign](https://www.npmjs.com/package/deep-assign)`npm install --save deep-assign`
1. 先说支持流程图
安装 `npm install --save hexo-filter-flowchart`
然后在hexo的_config.yml文件末尾新增如下配置：
```
flowchart:
  raphael:   https://cdnjs.cloudflare.com/ajax/libs/raphael/2.2.7/raphael.min.js
  flowchart: https://cdnjs.cloudflare.com/ajax/libs/flowchart/1.6.5/flowchart.min.js
  options:  drawSVG

```
新建一个文章键入以下内容：
```
# 引用代码后面加上flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e

```
会生成如下流程图
```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```
2. 再说支持时序图
安装`npm install --save hexo-filter-sequence`
然后在hexo的_config.yml文件末尾新增如下配置：
```
sequence:
  webfont: https://cdnjs.cloudflare.com/ajax/libs/webfont/1.6.27/webfontloader.js
  snap:    https://cdnjs.cloudflare.com/ajax/libs/snap.svg/0.4.1/snap.svg-min.js
  underscore:  https://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.8.3/underscore-min.js
  sequence:  https://cdnjs.cloudflare.com/ajax/libs/js-sequence-diagrams/1.0.6/sequence-diagram-min.js
  css: # optional, the url for css, such as hand drawn theme 
  options: 
    theme: simple
    css_class: 

```
新建一个文章键入以下内容：
```
# 引用代码后面加上sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```
但是却没有如愿生成时序图，根据[这篇博客](http://wewelove.github.io/fcoder/2017/09/06/markdown-sequence/)修改node_modules/hexo-filter-sequence/lib/renderer.js源码第25行`var config = this.config.flowchart;`为`var config = this.config.sequence;`重启hexo，会生成如下时序图
```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```
---
## 以下内容是搭建成功后Hexo的默认第一篇文章。作为命令的记录没有删除。

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



