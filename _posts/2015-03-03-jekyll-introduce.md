---
layout:		post
title:		Jekyll简介
category:	[Jekyll]
tags:		[Jekyll]
published:	true
---
# Jekyll简介
---

如果你只想快速搭建一个 github 的静态网站, 而暂时没有时间来研究 jekyll 语法的话,建议直接 fork 我的这个当然,阅读一下之前我记录的一些笔记也可以增长一些知识.

## Jekyll
jekyll 是一个静态网站生成器.
jekyll 通过 标记语言 markdown 或 textile 和 模板引擎 liquid 转换生成网页.
github 为我们提供了这个一个地方, 可以使用 jekyll 做一个我们自己的网站.

这里不介绍怎么在本地安装使用 jekyll, 如果你想在本地使用,请参考官方文档的 安装教程 和 使用教程.
不过这里可以透漏一下, jekyll 依赖于 ruby .

<!--break-->

## 目录

<pre>
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes/
|   ├── footer.html
|   └── header.html
├── _layouts/
├── _posts/
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site/
├── about/
|   └── index.html
├── contact/
|   └── index.html
└── index.html
</pre>

* **config.yml** 保存配置数据。很多配置选项都可以直接在命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。
* **_drafts drafts** 是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据。学习如何使用 drafts.
* **_includes** 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签  \{\% include file.ext \%\} 来把文件 _includes/file.ext 包含进来。
* **_layouts layouts** 是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择。 这将在下一个部分进行介绍。标签  { { content } } 可以将content插入页面中。
* **_posts** 这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。The permalinks 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。
* **_site** 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中。
* **index.html** 如果这些文件中包含 YAML 头信息 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 .html, .markdown, .md, 或者 .textile 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。
* **Other** 其他一些未被提及的目录和文件如 css 还有 images 文件夹，favicon.ico 等文件都将被完全拷贝到生成的 site 中。这里有一些使用 Jekyll 的站点，如果你感兴趣就来看看吧。

## 基本用法

安装了 Jekyll 的 Gem 包之后，就可以在命令行中使用 Jekyll 命令了。有以下这些用法：

<pre>
$ jekyll build
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中。

$ jekyll build --destination <destination>
# => 当前文件夹中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --source <source> --destination <destination>
# => 指定源文件夹<source>中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --watch
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中，
#    查看改变，并且自动再生成。
</pre>

Jekyll 同时也集成了一个开发用的服务器，可以让你使用浏览器在本地进行预览。

<pre>
$ jekyll serve
# => 一个开发服务器将会运行在 http://localhost:4000/

$ jekyll serve --detach
# => 功能和`jekyll serve`命令相同，但是会脱离终端在后台运行。
#    如果你想关闭服务器，可以使用`kill -9 1234`命令，"1234" 是进程号（PID）。
#    如果你找不到进程号，那么就用`ps aux | grep jekyll`命令来查看，然后关闭服务器。[更多](http://unixhelp.ed.ac.uk/shell/jobz5.html).

$ jekyll serve --watch
# => 和`jekyll serve`相同，但是会查看变更并且自动再生成。
</pre>

还有一些可以配置的配置选项. 很多配置选项既可以在命令行中作为标识(flags)设定，也可以在源文件根目录中的 _config.yml 文件中进行设定。Jekyll 会自动加载这些配置。比如你在你的 _config.yml 文件中添加了下面几行：

<pre>
source:      _source
destination: _deploy
</pre>

那么就等价于执行了以下两条命令：

<pre>
$ jekyll build
$ jekyll build --source _source --destination _deploy
</pre>

## 配置

Jekyll允许你很轻松的设计你的网站，这很大程度上归功于灵活强大的配置功能。既可以配置在网站根目录下的  _config.yml 文件，也可以作为命令行的标记来配置。

### 全局配置

| 名称              | 描述                              |  选项 和 标记               |
| ------------------|:---------------------------------:|:----------------------------|
|  Site Source      |  修改 Jekyll 读取文件的路径       |  source: DIR                |
|  Site Destination |  修改 Jekyll 写入文件的路径       |  destination: DIR           |
|  Safe             |  禁用 自定义插件。                |  safe: BOOL                 |
|  Exclude          |  转换时排除某些文件、文件夹       |  exclude: [DIR, FILE, ...]  |
|  Include          |  转换时强制包含某些文件、文件夹   |  include: [DIR, FILE, ...]  |
|  Time Zone        |  设置时区                         |  timezone: TIMEZONE         |
|  Encoding         |  设置文件的编码                   |  encoding: ENCODING         |

### 编译选项

| 名称              | 描述                              |  选项 和 标记               |
| ------------------|:---------------------------------:|:----------------------------|
|  Regeneration     |  允许文件修改时自动重新生成网站   |  -w, --watch                |
|  Configuration    |  手动设置配置文件，可设置多个     |  --config FILE1[,FILE2,...] |
|  Drafts           |  处理草稿                         |  --drafts                   |
|  Future           |  为相关文章生成索引               |  future: BOOL               |
|  LSI              |  允许文件修改时自动重新生成网站   |  lsi: BOOL                  |
|  Limit Posts      |  限制文章的数量                   |  limit_posts: NUM           |

### 服务选项

除了下边的选项， serve 命令还可以接收 build 的选项，当运行网站服务之前的编译时候使用。

| 名称                   | 描述                    |  选项 和 标记   |
| -----------------------|:-----------------------:|:----------------|
|  Local Server Port     |  监听所给的端口         |  port: PORT     |
|  Local Server Hostname |  监听所给的主机名       |  host: HOSTNAME |
|  Base URL              |  网站的根路径           |  baseurl: URL   |
|  Detach                |  从终端命令行中分离出来 |  detach: BOOL   |

## 变量
所有的变量是都一个树节点, 比如模板中定义的头部变量,需要使用下面的语法获得

### 全局变量

* **site** site 的信息 以及 _config.yml 文件中的配置信息
* **page** 页面的配置信息
* **content** 模板中,用于引入子节点的内容
* **paginator** 分页信息(一旦paginate配置选项被设置了，这个变量才能被使用。)

### site 下的变量

* **site.time** 运行 jekyll 的时间
* **site.pages** 所有页面
* **site.posts** 所有文章
* **site.related_posts** 类似的10篇文章,默认最新的10篇文章,指定lsi为相似的文章
* **site.static_files** 没有被 jekyll 处理的文章,有属性 path, modified_time 和 extname.
* **site.html_pages** 所有的 html 页面
* **site.collections** 新功能,没使用过
* **site.data _data** 目录下的数据
* **site.documents** 所有 collections 里面的文档
* **site.categories** 所有的 categorie
* **site.tags** 所有的 tag
* **site.[CONFIGURATION_DATA]** 自定义变量

### page 下的变量

* **page.content** 页面的内容
* **page.title** 标题
* **page.excerpt** 摘要
* **page.url** 链接
* **page.date** 时间
* **page.id** 唯一标示
* **page.categories** 分类
* **page.tags** 标签
* **page.path** 源代码位置
* **page.next** 下一篇文章
* **page.previous** 上一篇文章

### paginator 下的变量

* **paginator.per_page** 每一页的数量
* **paginator.posts** 这一页的数量
* **paginator.total_posts** 所有文章的数量
* **paginator.total_pages** 总的页数
* **paginator.page** 当前页数
* **paginator.previous_page** 上一页的页数
* **paginator.previous_page_path** 上一页的路径
* **paginator.next_page** 下一页的页数
* **paginator.next_page_path** 下一页的路径

## 参考

* [Jekyll 模板 简单语法 笔记](http://github.tiankonguse.com/blog/2014/09/26/jekyll-base-record/)
* [在github上制作一个静态网站](http://github.tiankonguse.com/blog/2014/07/10/make-github-website/)
* [Jekyll官方网站](http://jekyllrb.com/)
* [Jekyll中文网站](http://jekyllcn.com/)