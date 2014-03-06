#Command line tool
新的0.10版本.

Scrapy是通过Scrapy的命令行工具进行控制,在这里我们简称为"Scrapy 工具",为了从他们的子指令中区分开来,我们将其称为"命令"或者"Scrapy 命令."

Scrapy 工具提供了一些命令,为了不同的目的,每一条指令都可以接受不同的参数和选项.

##Scrapy项目的默认结构


在深入研究的命令行工具和它的子命令之前，让我们先来了解一个Scrapy项目的目录结构。


虽然Scrapy可以被我们进行修改,但是所有的Scrapy项目都有相同的默认文件结构,和下面相似:

	scrapy.cfg
	myproject/
    	__init__.py
    	items.py
    	pipelines.py
    	settings.py
    spiders/
        __init__.py
        spider1.py
        spider2.py
        ...
        

scrapy.cfg文件所在的目录是该项目的目录.该文件包含了定义项目设置的python模块.下面是一个例子.

	[settings]
	default = myproject.settings
	

##使用Scrapy工具

可以从通过不带参数运行Scrapy工具开始,它会输出一些用法说明和可用的命令：

	Scrapy X.Y - no active project

	Usage:
  		scrapy <command> [options] [args]

	Available commands:
  		crawl         Run a spider
  		fetch         Fetch a URL using the Scrapy downloader
		[...]

如果你是一个Scrapy项目中运行这条指令,第一行会输出当前的活动项目.在上面的情况中,它是从一个项目之外运行的.如果从一个项目里面运行它的话,会输出这样的信息：

	Scrapy X.Y - project: myproject

	Usage:
  		scrapy <command> [options] [args]

		[...]

##创建新项目

通常,利用scrapy工具你会做的第一件事就是创建属于你的Scrapy项目：

	scrapy startproject myproject
	
这将会在myproject目录下面建立一个Scrapy项目.

接下来,打开新项目的目录.

	cd myproject
	
你将会在这里用Scrapy的命令来管理和控制你的项目.

##项目控制

使用项目里面的Scrapy工具来控制和管理项目.

举个例子,新建一个Spider.

	scrapy genspider mydomain mydomain.com


有些Scrapy命(爬crawl)必须从Scrapy项目内运行.请参阅下面的命令,了解有关的必须在项目内运行的命令和不必在项目内运行的命令的详细信息.

请记住,当在项目内运行一些命令时,它们可能会产生略有不同的行为.例如,fetch指令将使用Spider被重写的行为(如user_agent属性来重写user_agent)如果被获取的url与某些特定的Spider有关.这是故意的,因为fetch指令的目的是用来检查Spider是如何在下载网页.

##可用的工具命令
这一部分包含了一份关于可用的内置命令的描述和一些用法示例的列表.记住,你总是可以通过运行下面这条指令来获取有关每个命令的详细信息：

	scrapy <command> -h
	
你可以使用下面这条指令,查看所有可用的指令:
	
	scrapy -h

Scrapy中有两种类型的命令，即只能在一个Scrapy项目内部工作的指令(项目特定的指令)，和那些在没有活动的Scrapy项目时依旧可以运行的指令(全局指令),虽然它们在一个项目内部运行时，行为可能稍有不同(因为他们将使用项目重写设置).

全局指令

	startproject
	settings
	runspider
	shell
	fetch
	view
	version

项目内指令

	crawl
	check
	list
	edit
	parse
	genspider
	deploy
	bench
	
	
####startproject

语法:scrapy startproject <project_name>

是否需要项目:否

在project_name的目录下,创建一个名叫project_name的Scrapy项目.

使用范例:

	$ scrapy startproject myproject
	
	
####genspider
语法: scrapy genspider [-t template] <name> <domain>

是否需要项目: 是

在当前项目中创建一只新Spider.


这仅仅是基于预先定义的模板创建Spider的快捷指令,但肯定不是创建Spider的唯一途径.你可以仅仅创建Spider的源代码文件,而不是使用这一条指令.

使用范例:

	$ scrapy genspider -l
	Available templates:
  		basic
  		crawl
  		csvfeed
  		xmlfeed

	$ scrapy genspider -d basic
	from scrapy.spider import Spider

	class $classname(Spider):
    	name = "$name"
    	allowed_domains = ["$domain"]
    	start_urls = (
        	'http://www.$domain/',
        	)

    	def parse(self, response):
        	pass

	$ scrapy genspider -t basic example example.com
	Created spider 'example' using template 'basic' in module:
  		mybot.spiders.example
  	
####crawl
语法: scrapy crawl <spider>

是否需要项目: 是

开始用Spider抓取.

使用范例:

	$ scrapy crawl myspider
	[ ... myspider starts crawling ... ]
####check
语法: scrapy check [-l] <spider>

是否需要项目: 是

开始检查

使用范例:

	$ scrapy check -l
	first_spider
  	 * parse
  	 * parse_item
	second_spider
  	 * parse
  	 * parse_item

	$ scrapy check
	[FAILED] first_spider:parse_item
	>>> 'RetailPricex' field is missing

	[FAILED] first_spider:parse
	>>> Returned 92 requests, expected 0..4
	
####list
语法: scrapy list

是否需要项目: 是

列出当前项目中所有可用的Spider.输出为每行显示一只Spider.

使用范例:

	$ scrapy list
	spider1
	spider2
####edit
语法: scrapy edit <spider>

是否需要项目: yes

使用EDITOR设置中定义的编辑器编辑所给的Spider.

这一条指令仅仅只是被用作普遍情况下的快捷方式,开发者可以使用任何或者IDE来开发或者debug他的Spiders.

使用范例:

	$ scrapy edit spider1
####fetch
语法: scrapy fetch <url>

是否需要项目: 否

使用Scrapy下载器下载给定的URL和并且将结果写入标准输出.

关于这条指令的一件很有趣的事情是,它获取的页面怎么蜘蛛会下载它.例如，如果Spider有重写的user_agent属性,它将会使用该属性.

所以这条指令可以用来“看”你的Spider是如何抓取某些页面.

如果在一个项目外使用这一条指令,没有特别的Spider行为将被应用,它将只使用默认的Scrapy下载器设置.

使用范例:

	$ scrapy fetch --nolog http://www.example.com/some/page.html
	[ ... html content here ... ]

	$ scrapy fetch --nolog --headers http://www.example.com/
	{'Accept-Ranges': ['bytes'],
 	'Age': ['1263   '],
 	'Connection': ['close     '],
 	'Content-Length': ['596'],
	'Content-Type': ['text/html; charset=UTF-8'], 	'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
 	'Etag': ['"573c1-254-48c9c87349680"'],
 	'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
 	'Server': ['Apache/2.2.3 (CentOS)']}
####view
语法: scrapy view <url>

是否需要项目: 否


在浏览器中用你的Scrapy Spider能够“看到”的方式来打开给定的URL.有时候,Spider看到的页面不同于普通用户看到的页面,所以这可以被用来检查Spider“看到”了什么并确认它是否是你所期望的结果.

使用范例:

	$ scrapy view http://www.example.com/some/page.html
	[ ... browser starts ... ]
####shell
语法: scrapy shell [url]

是否需要项目: 否

为给定的URL,启动Scrapy shell,或者在不给定URL的情况时,为空.

更多详细信息请查看Scrapy shell.

使用范例:

	$ scrapy shell http://www.example.com/some/page.html
	[ ... scrapy shell starts ... ]
####parse
语法: scrapy parse <url> [options]

是否需要项目: yes

利用处理它的Spider来获取给定的URL并解析页面,通过与 --callback 选项使用该方法，或在未给出选项的时候解析它.

支持的选项：

* --callback or -c: 用于解析响应的Spider回调方法

* --rules or -r: 用CrawlSpider规则发现用于解析响应的会回调方法(ie. spider method)
* 
* --noitems: 不显示抓取的内容

* --nolinks: 不显示提取的链接

* --depth or -d: 每个请求应该地柜遵循的深度级别.(默认为1)

* --verbose or -v: 显示每个深度级别的信息

使用范例:

	$ scrapy parse http://www.example.com/ -c parse_item
	[ ... scrapy log lines crawling example.com spider ... ]

	>>> STATUS DEPTH LEVEL 1 <<<
	# Scraped Items  ------------------------------------------------------------
	[{'name': u'Example item',
 	'category': u'Furniture',
 	'length': u'12 cm'}]

	# Requests  -----------------------------------------------------------------
	[]
####settings
语法: scrapy settings [options]

是否需要项目: 否

获取Scrapy设置的值

如果在项目里面使用它,会显示该项目的设定值,否则它会显示默认Scrapy的设置值.

用法范例:

	$ scrapy settings --get BOT_NAME
	scrapybot
	$ scrapy settings --get DOWNLOAD_DELAY
	0
####runspider
语法: scrapy runspider <spider_file.py>

是否需要项目: 否

在不用创建一个项目的情况下,运行包含在Python文件中的Spider.

Example usage:

用法范例:

	$ scrapy runspider myspider.py
	[ ... spider starts crawling ... ]
####version

语法: scrapy version [-v]

是否需要项目: 否

输出Scrapy的版本信息,如果加入-v参数,它还会输出Python,Twisted和Platform的版本信息,这对bug reports是很有用的.

####deploy

新的0.11版本.

语法: scrapy deploy [ <target:project> | -l <target> | -L ]

是否需要项目: 是

部署项目到Scrapyd服务器.请参阅[Deploying your project](http://scrapyd.readthedocs.org/en/latest/#deploying-your-project).

####bench
New in version 0.17.

新的0.17版本.

语法: scrapy bench

是否需要项目: 否

快速运行基准测试.[Benchmarking](http://scrapyd.readthedocs.org/en/latest/#deploying-your-project).

##自定义项目指令


你也可以通过使用COMMANDS_MODULE设置添加自定义的项目指令.请在[scrapy/commands](https://github.com/scrapy/scrapy/tree/master/scrapy/commands)中产看命令就如何实施你的指令的例子.

####COMMANDS_MODULE
	Default: '' (empty string)

一个用于寻找自定义的Scrapy指令的模块.被用于在你的Scrapy项目中添加自定义的指令.

Example:
范例:

	COMMANDS_MODULE = 'mybot.commands'