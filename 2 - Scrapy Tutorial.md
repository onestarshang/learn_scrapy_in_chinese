#Scrapy Tutorial#

###Scrapy 指南###

在这个指南中我们假定你已经在你的系统上安装好了Scrapy，如果没有请看安装指南。

我们将用Open directory project (dmoz)作为示例。

这个指南会让你了解一下几项：

- 创建一个新的Scrapy项目
- 定义你要抓取的Items
- 写爬虫类scrapy和解析Items
- 写一个存储items的管道（Itme Pipeline）

Scrapy使用Python写的，如果你是python新手，为了更加了解Scrapy，需要先学习python。如果你熟悉其他的编程语言，推荐Learn Python The Hard Way。如果你想从学习python开始学习编程，可以看一下this list of Python resources for non-programmers。

###创建项目###

在爬站之前，需要先创建一个新的Scrapy项目，输入你想要存放代码的目录名然后运行：

    scrapy startproject tutorial
这将创建一个包含以下内容的tutorial目录：

    tutorial/
        scrapy.cfg
    	tutorial/
       		__init__.py
        	items.py
        	pipelines.py
        	settings.py
        	spiders/
            	__init__.py
            	...
这些是必备的：



- scrapy.cfg：项目配置文件
- tutorial/：项目的python模块，之后你将从这里import你的代码
- tutorial/ items.py：项目的itmes文件
- tutorial/ pipelines.py：项目的pipelines文件
- tutorial/ settings.py：项目的设置文件
- tutorial/ spiders/：之后放置spider文件的目录

###定义我们的Item###

Items是可以被加载的容器，它存储爬取来的数据；它想python中的dict一样简单，但是可以对未声明的字段提供额外保护，以防止错误字符。

声明Items需要创建一个scrapy.item.Item类，通过scrapy.item.Field类定义Items的属性。

我们通过dmoz.org上包含的数据来创建一个item，由于我们需要的是站点的名称，url和描述，因此为这三个属性定义fields。在tutorial下编辑items.py，Items类的如下：

    from scrapy.item import Item, Field
    class DmozItem(Item):
    	title = Field()
    	link = Field()
    	desc = Field()
一开始这看上去很复杂，但是定义item有助于利用Scrapy上其他组件，所以你需要知道你的item是什么样的。

###第一个Spider类###

spider是用来爬站的用户自己写的类。

spider定义了一个初始化好的需要下载的URLs的list，如何跟踪链接的规则，如何为提取items解析网页上的内容的方法。

创建一个Spider，需要继承scrapy.spider.BaseSpider类，并且定义三个主要的必不可少的属性：

- name：定义了Spider类，必须唯一，不能在多个Spider中定义相同的名称。
- start_urls：Spider开始爬站的一组URLs的list。所以，一开始需要下载的页面链接在这里定义，随后的一些链接都陆续从start_urls包含的数据中产生。
- parse()是Spider的一个方法，它可以调用每个start URLs返回的Response对象。这个对象作为方法的第一个也是唯一的一个参数值。
这个方法负责处理response并返回提取数据，还有页面中更多的URLs。

	<pre><code>from scrapy.spider import BaseSpider
	class DmozSpider(BaseSpider):
        name = "dmoz"
        allowed_domains = ["dmoz.org"]
        start_urls = [
                 "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
                 "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
                 ]
 
        def parse(self, response):
        	filename = response.url.split("/")[-2]
        	open(filename, 'wb').write(response.body)</code></pre>
###抓取###

想让spider工作，到最上层目录，然后运行：

scrapy crawl dmoz
crawl dmz可以让spider抓取dmoz.org域名，你可以看到类似以下的输出：
<pre><code>2008-08-20 03:51:13-0300 [scrapy] INFO: Started project: dmoz
2008-08-20 03:51:13-0300 [tutorial] INFO: Enabled extensions: ...
2008-08-20 03:51:13-0300 [tutorial] INFO: Enabled downloader middlewares: ...
2008-08-20 03:51:13-0300 [tutorial] INFO: Enabled spider middlewares: ...
2008-08-20 03:51:13-0300 [tutorial] INFO: Enabled item pipelines: ...
2008-08-20 03:51:14-0300 [dmoz] INFO: Spider opened
2008-08-20 03:51:14-0300 [dmoz] DEBUG: Crawled <http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/> (referer: None)
2008-08-20 03:51:14-0300 [dmoz] DEBUG: Crawled <http://www.dmoz.org/Computers/Programming/Languages/Python/Books/> (referer: None)
2008-08-20 03:51:14-0300 [dmoz] INFO: Spider closed (finished)</code></pre>
注意包含[dmoz]的几行，这个是你的spider的输出，你可以看到很长的那行，就是在start_urls中定义的URLs。因为是起始URLs，所以并没有引用，他们在这一行的最后显示：(referer: <None>)。

**在引擎内发生了什么？**

Scrapy为spider中的每个定义在start_urls中的URLs创建一个scrapy.http.Request对象，并将他们传递给sipder中作为回调函数的parse()方法。

这些请求对象被调度和执行，再然后通过spider的parse()方法返回scrapy.http.Response对象。

###提取Items###

**介绍Selectors**

有几种方法提取页面中的数据，Scrapy用的机制是基于XPath表达式的XPath selectors。想了解更多关于selectors信息和其他机制，请参考XPath selectors documentation。

这里有一些XPath的例子和他们所代表的含义：

- /html/head/title：选择在HTML页面中查找heda标签下的title标签元素
- /html/head/title/text()：选择上述的title标签中的文本内容
- //td：选择所有td标签
- //div[@class="mine"]：选择所有div标签中，class为mine的元素
这里只是一些用XPath表达式的简单例子，但是XPath表达式的功能确实比这些更加强大，学习XPath表达式推荐this XPath tutorial。

为了能用XPath，Scrapy提供了XPathSelector类，它包括HtmlXPathSelector和XmlXPathSelector。为了用他们，实例化的时候必须传递Response对象参数。

你可以看到selectors对象可是表示文档中的节点。所以初始化的selectors是文档的根节点，或者是整个文档。

Selectors有三个方法：

- select()：返回一个selectors的list，每个都表示它是根据传入的XPath表达式想要找的节点
- extract()：返回unicode字符串（selector通过XPath选取的数据）
- re()：返回根据传入的正则表达式选取的字符串list
通过Shell运行Selectors

为了解释Selector，我们用内置的Scrapy Shell功能，它需要iPython。

进入到项目的最上层目录并运行：

    scrapy shell "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/"
看到的输出：

<pre><code>[ ... Scrapy log here ... ]
 
[s] Available Scrapy objects:
[s] 2010-08-19 21:45:59-0300 [default] INFO: Spider closed (finished)
[s] hxs <HtmlXPathSelector (http://www.dmoz.org/Computers/Programming/Languages/Python/Books/) xpath=None>
[s] item Item()
[s] request <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
[s] response <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
[s] spider <BaseSpider 'default' at 0x1b6c2d0>
[s] xxs <XmlXPathSelector (http://www.dmoz.org/Computers/Programming/Languages/Python/Books/) xpath=None>
[s] Useful shortcuts:
[s] shelp() Print this help
[s] fetch(req_or_url) Fetch a new request or URL and update shell objects
[s] view(response) View response in a browser
 
In [1]:</code></pre>
当shell加载后，你可以得到一个本地response变量的响应，如果你输入response.body，你可以看到页面文本，如果你输入response.headers，你可以看到页面headers。

shell实例化了两个selectors，一个是HTML的，一个是XML的。

试一下：

<pre><code>In [1]: hxs.select('//title')
Out[1]: [<HtmlXPathSelector (title) xpath=//title>]
 
In [2]: hxs.select('//title').extract()
Out[2]: [u'<title>Open Directory - Computers: Programming: Languages: Python: Books</title>']
 
In [3]: hxs.select('//title/text()')
Out[3]: [<HtmlXPathSelector (text) xpath=//title/text()>]
 
In [4]: hxs.select('//title/text()').extract()
Out[4]: [u'Open Directory - Computers: Programming: Languages: Python: Books']
 
In [5]: hxs.select('//title/text()').re('(\w+):')
Out[5]: [u'Computers', u'Programming', u'Languages', u'Python']</code></pre>
###提取数据###

现在我们从页面中提取真正的数据。

你可以在终端输入response.body，然后拼出想要找的数据的XPath表达式。然而，查看源码非常枯燥，可以用FireFox的FireBug插件。

看完源码，你发现网站信息在ul标签中，实际上是第二个ul标签。

我们像这样可以选择属于每个网站地址列表标签li：

    hxs.select('//ul/li')
根据它找到描述信息：

    hxs.select('//ul/li/text()').extract()
网站标题：

    hxs.select('//ul/li/a/text()').extract()
网站链接：

    hxs.select('//ul/li/a/@href').extract()
之前已经说过了，每个select()返回selectors的list，所以我们可以向下一层叠调用select()方法。

可以这样用：

<pre><code>sites = hxs.select('//ul/li')
for site in sites:
    title = site.select('a/text()').extract()
    link = site.select('a/@href').extract()
    desc = site.select('text()').extract()
    print title, link, desc</code></pre>
将代码添加到spider类中：

<pre><code>from scrapy.spider import BaseSpider
from scrapy.selector import HtmlXPathSelector
 
class DmozSpider(BaseSpider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
        ]
 
    def parse(self, response):
        hxs = HtmlXPathSelector(response)
        sites = hxs.select('//ul/li')
        for site in sites:
            title = site.select('a/text()').extract()
            link = site.select('a/@href').extract()
            desc = site.select('text()').extract()
            print title, link, desc</code></pre>
###添加Items###

items像用户定义的python字典，你可以用pyhon字典的标准语法通过指定key获取对应的值。

<pre><code>>>>item = DmozItem()
>>>item['title'] = 'Example title'
>>>item['title']
'Example title'</code></pre>
Spider类也可以返回包含提取好的数据的item类。可以这样获取让spider返回items对象：

<pre><code>from scrapy.spider import BaseSpider
from scrapy.selector import HtmlXPathSelector
 
from tutorial.items import DmozItem
 
class DmozSpider(BaseSpider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
        ]
 
    def parse(self, response):
        hxs = HtmlXPathSelector(response)
        sites = hxs.select('//ul/li')
        items = []
        for site in sites:
            item = DmozItem()
            item['title'] = site.select('a/text()').extract()
            item['link'] = site.select('a/@href').extract()
            item['desc'] = site.select('text()').extract()
            items.append(item)
        return items</code></pre>
现在你可以爬取dmoz.org的数据了：

<pre><code>[dmoz] DEBUG: Scraped from <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
{'desc': [u' - By David Mertz; Addison Wesley. Book in progress, full text, ASCII format. Asks for feedback. [author website, Gnosis Software, Inc.\n],
'link': [u'http://gnosis.cx/TPiP/'],
'title': [u'Text Processing in Python']}
[dmoz] DEBUG: Scraped from <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
{'desc': [u' - By Sean McGrath; Prentice Hall PTR, 2000, ISBN 0130211192, has CD-ROM. Methods to build XML applications fast, Python tutorial, DOM and SAX, new Pyxie open source XML processing library. [Prentice Hall PTR]\n'],
'link': [u'http://www.informit.com/store/product.aspx?isbn=0130211192'],
'title': [u'XML Processing with Python']}</code></pre>
###存储爬取的数据###

最简单的方法是用Feed Exports存储提取的数据，像这样：

    scrapy crawl dmoz -o items.json -t json
正将创建一个items.json文件，里面是返回的item的json序列化字符串。

像一些小项目，这可能足够了，如果你想要展示更加复杂的数据，可以写一个Item Pipeline。想Items一样，创建项目的时候，Pipelines文件也被创建了。如果你只是想要存储爬取的item数据，也用不到pipeline。
