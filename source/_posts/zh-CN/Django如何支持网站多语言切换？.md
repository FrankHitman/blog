---
title: Django如何支持网站多语言切换？
date: 2018-04-08 19:11:31
tags: Django
---

## 分为models， views和模版（templates）三处的英文转化。

##### 首先修改settings
1.中间件中增加一个django.middleware.locale.LocaleMiddleware的中间件，并且它的顺序要靠前，笔者的是放第二个。

![settings中间件配置截图](http://upload-images.jianshu.io/upload_images/1909919-9f966f52ba3d9bab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.增加LANGUAGES和LOCALE_PATHS，并且手动在项目根目录下创建locale文件夹
![增加LANGUAGES和LOCALE_PATHS](http://upload-images.jianshu.io/upload_images/1909919-6cd98289baffddb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
***
<!--more-->
##### 修改template目录下的html文件
1.每一个html前面都要加上 ` load i18n `，当然如果嫌麻烦，可以把它放在已经存在公共页面里面。
2.在需要翻译的地方加上` trans ‘XXXXX’ `，XXXXX是需要翻译的文本

![ trans ‘XXXXX’ ](http://upload-images.jianshu.io/upload_images/1909919-de7b25c4a2a8ae1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.多次重复翻译的内容可以像设置常量那样
```html
{% trans "This is the title" as the_title %}
<title>{{ the_title }}</title>
<meta name="description" content="{{ the_title }}">
```
4.如果翻译的内容有django模板输出的变量，就需要用`blocktrans`和`endblocktrans`
```python
# 普通用法
{% blocktrans %}This string will have {{ value }} inside.{% endblocktrans %}
# 增加过滤器
{% blocktrans with amount=article.price %}
That will cost $ {{ amount }}.
{% endblocktrans %}
{% blocktrans with myvar=value|filter %}
This will have {{ myvar }} inside.
{% endblocktrans %}
# 多个过滤器
{% blocktrans with book_t=book|title author_t=author|title %}
This is {{ book_t }} by {{ author_t }}
{% endblocktrans %}
# 注意：其他bolck tags (例如 {% for %} or {% if %}) 不允许在 blocktrans tag内部.
```
***
##### 生成翻译文件，并且翻译
1.执行如下命令生成翻译文件
```
python manage.py makemessages -l en
# en 指的是English，其他以此类推
```
接下来看到local文件夹下面生成了en/LC_MESSAGES/
![local/en/LC_MESSAGES/](http://upload-images.jianshu.io/upload_images/1909919-c5ee0a69d6d4ddb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.修改django.po文件，对每一个msgid进行翻译，翻译内容填在msgstr中。
![修改django.po文件](http://upload-images.jianshu.io/upload_images/1909919-1dab8307a50f6987.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.然后编译
```shell
python manage.py compilemessages -l en
```

##### 修改url，此url文件
```python
# 之前的url
urlpatterns = patterns('',)
# 修改后
from django.conf.urls.i18n import i18n_patterns
urlpatterns += i18n_patterns('',)
```
##### 重启服务并访问查看效果
http://localhost:端口号/en/adminbd/ap_list
http://localhost:8080/zh-hans/adminbd/ap_list
****
##### 最后来添加切换语言的功能
1.添加切换语言的url：
```
url(r'^i18n/',include('django.conf.urls.i18n')),
```
2.页面上添加如下的form表单（必须是POST请求）

![html](http://upload-images.jianshu.io/upload_images/1909919-acbe9f1fe7f7405d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还有JavaScript代码

![切换语言JavaScript代码](http://upload-images.jianshu.io/upload_images/1909919-63b2f2cd5a88ac95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
****
##### 如果涉及到登录的系统，需要记住用户选择的语言，那么就需要存储session，session名为"_language"
```
request.session['_language']='zh-hans'
```
##### views的多语言，具体可以参考[官方文档](https://docs.djangoproject.com/es/1.9/topics/i18n/translation/#standard-translation)
```
from django.utils.translation import ugettext as _
#……
MSG={1:'成功'，0:'失败'}
context['msg']=_(MSG[1])
return render(request,RESPONSE_URL,context)
```
****

##### 关于javascript的国际化
[官方文档](https://docs.djangoproject.com/es/1.9/topics/i18n/translation/#module-django.views.i18n)
在根urls.py 中添加如下代码，your.app.package替换成app名称
```
from django.views.i18n import javascript_catalog
js_info_dict = { 'packages': ('your.app.package',),}
urlpatterns ＋= [ url(r'^jsi18n/$', javascript_catalog, js_info_dict, name='javascript-catalog'),]
```

![配置JavaScript的url](http://upload-images.jianshu.io/upload_images/1909919-22f707af2499f43b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在模板中先于需要汉化js引用如下js
```html
<script type="text/javascript" src="{% url 'javascript-catalog' %}"></script>
```
1.然后在使用gettext方法标记需要国际化的内容

![539181.png](http://upload-images.jianshu.io/upload_images/1909919-0d22523c6d8cad1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.涉及到单数复数形式的用 ngettext
var object_count = 1 // or 0, or 2, or 3, ...s = ngettext('literal for the singular case’, //单数 'literal for the plural case’, object_count);//复数

3.涉及到格式化输出内容的用interpolate，与ngettext搭配使用
fmts = ngettext('There is %s object. Remaining: %s', 'There are %s objects. Remaining: %s', 11);s = interpolate(fmts, [11, 20]);// s 输出结果为为复数形式 'There are 11 objects. Remaining: 20'

还有其他方法
* get_format
* gettext_noop
* pgettext
* npgettext
* pluralidx

最后与makemessages一样，js也需要生产翻译文件，得到djangojs.po文件，在里面添加翻译
```
django-admin makemessages -d djangojs -l en
```

编译还是与上面的命令一样。
重启服务器可以看到效果
***
##### 着重强调一点，在JavaScript多语言这里遇到个坑，js的翻译不管是中文还是英文都是显示英文翻译。原因是变异的时候应该用 *zh_Hans*  而不是zh_hans 或者zh_cn等等。
```
  django-admin.py makemessages -l zh_Hans 
```
上面图中也是错的

![93438.png](http://upload-images.jianshu.io/upload_images/1909919-18d76c414e4b66c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
***
##### 最后一个坑是多语言国际化的时候渲染模板的方法要用render（request，’xxxxx.html’,{}）而不是render_to_response（’xxxx.html’，｛｝）方法
***
[中文文档](http://djangobook.py3k.cn/2.0/chapter19/)

作为默认， `django-admin.py makemessages`检测每一个有 .html
 扩展名的文件。  以备你要重载缺省值，使用 --extension或 -e选项指定文件扩展名来检测。
```python
django-admin.py makemessages -l de -e txt
```
用逗号和（或）使用-e或--extension来分隔多项扩展名：
```python
django-admin.py makemessages -l de -e html,txt -e xml
```
当创建JavaScript翻译目录时，你需要使用特殊的Django域：**not** -e js
<span id="busuanzi_container_page_pv">
  本文总阅读量<span id="busuanzi_value_page_pv"></span>次
</span>
