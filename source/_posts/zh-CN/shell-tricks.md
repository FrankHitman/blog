---
title: shell tricks
lang: zh-CN
date: 2018-04-17 11:19:36
categories:
    - linux
tags: shell
---
##### 一些可能你不知道的 shell 用法和脚本，简单 & 强大
1. `sudo !!`
 以root的身份执行上一条命令 。
 场景举例：比如Ubuntu里用apt-get安装软件包的时候是需要root身份的，
我们经常会忘记在apt-get前加sudo。每次不得不加上sudo再重新键入这
行命令，这时可以很方便的用sudo !!完事。
<!-- more -->
2. `cd –`
 回到上一次的目录 。
 场景举例：当前目录为/home/a，用cd ../b切换到/home/b。这时可以通
过反复执行cd –命令在/home/a和/home/b之间来回方便的切换。
3. `^old^new`
 替换前一条命令里的部分字符串。
 场 景：echo "wanderful"， 其 实 是 想 输 出echo "wonderful"。 只需要
^an^on就行了，对很长的命令的错误拼写有很大的帮助。如果^a^o那么就会替换echo为echa
4. `man ascii`
 显示ascii码表。
 场景：忘记ascii码表的时候还需要google么?尤其在天朝网络如此“顺畅”
的情况下，就更麻烦在GWF多应用一次规则了，直接用本地的man ascii吧。
5. `ctrl-x e`
 快速启动你的默认编辑器（由变量$EDITOR设置）。这一条目前我在centos7环境下没有实验成功，是ctrl+x键按下然后按下e键吗？
修改默认编辑器为vim`echo export EDITOR=/usr/bin/vim >> ~/.bashrc`
6. `netstat –tlnp`
 列出本机进程监听的端口号。
7. `tail -f /path/to/file.log | sed '/^Finished: SUCCESS$/ q'`
 当file.log里出现Finished: SUCCESS时候就退出tail，这个命令用于实时
监控并过滤log是否出现了某条记录。这一条测试没有成功`tail -f /home/sdy/blog/blog/source/_posts/zh-CN/shell-tricks.md | sed '/^cd$/ q'`
8. `ssh user@server bash < /path/to/local/script.sh`
 在远程机器上运行一段脚本。这条命令最大的好处就是不用把脚本拷到远
程机器上。
9. `screen -d -m -S some_name ping my_router`
 后台运行一段不终止的程序，并可以随时查看它的状态。-d -m参数启动“分离”模式，-S指定了一个session的标识。可以通过-R命令来重新“挂载”一个标识的session。Screen是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。Screen中有会话的概念，用户可以在一个screen会话中创建多个screen窗口，在每一个screen窗口中就像操作一个真实的telnet/SSH连接窗口那样。更多细节请参考screen用法 man screen。[如何解决断开ssh连接后会自动终止正在执行的命令](https://www.ibm.com/developerworks/cn/linux/l-cn-screen/)
10. `wget --random-wait -r -p -e robots=off -U mozilla http://www.example.com`
 下载整个www.example.com网站。
11. `curl ifconfig.me`
 当你的机器在内网的时候，可以通过这个命令查看外网的IP。其实还有很多其他的域名网址用于查询ip，我用的是长城的宽带，发现好多获取的ip地址是不同的，后来了解到可能是长城租用其他运营商的网络，然后通过不同端口来区分用户，共享带宽。
```
curl icanhazip.com
curl ifconfig.me
curl curlmyip.com
curl ip.appspot.com
curl ipinfo.io/ip
curl ipecho.net/plain
curl www.trackip.net/i
```
12.`lsof –i`
实时查看本机网络服务的活动状态。
13.`python -m SimpleHTTPServer`
 一句话实现一个HTTP server，把当前目录设为HTTP服务目录，可以通
过http://localhost:8000访问 这也许是这个星球上最简单的HTTP服务器
的实现了。
14.最后来个复杂的——`history | awk '{CMD[$2]++;count++;} END { for (a in CMD )print CMD[a] " " CMD[a]/count*100 "%" a }' | grep -v "./" | column -c3 -s " " -t | sort -nr | nl | head -n10`
 这行脚本能输出你最常用的十条命令，由此甚至可以洞察你是一个什么
类型的程序员。
```
     1	522  52.2%git
     2	86   8.6%cd
     3	74   7.4%ls
     4	49   4.9%sudo
     5	36   3.6%pwd
     6	36   3.6%hexo
     7	20   2%npm
     8	19   1.9%vi
     9	15   1.5%who
    10	12   1.2%ssh

```
看不懂行代码？没关系，系统的学习一下*nix shell脚
本吧，力荐[《Linux命令行与Shell脚本编程大全》](http://www.ituring.com.cn/book/980)。

##### 参考资料[《码农增刊·Linus与Linux》](http://www.ituring.com.cn/book/1501)




