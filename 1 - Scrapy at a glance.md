#Scrapy at a glance#

###概览###

Scrapy 是一个可以抓取网站和从中提取结构化数据的应用框架，它可以应用于各种有用的应用中，比如数据挖掘，信息处理和历史归档。

虽然Scrapy最初是为了网站抓取而设计的，但是它也可以通过API（例如 Amazon Associates Web Services）提取数据或者用于更加一般目的的网站爬虫。

这个文档可以让你了解Scrapy的一般概念，知道它是如何工作的并且帮助你了解它是不是正如你需要的那样工作。

###选择一个网站###

如果你需要网站上的数据，但是它又没有提供相应的API或者获取信息的机制，scrapy可以帮助你获取信息。

比如说我们想要获取网站Mininova上today页面下的torrent种子的链接，名称，描述和大小信息，我们就可以直接从[http://www.mininova.org/today](http://www.mininova.org/today)页面中获取信息。

###定义你想获取的数据###

首先你需要定义你要获取的数据，在Scrapy中，Scrapy Items为你做好了一切。

以下是我们定义的Item：

    from scrapy.item import Item, Field
 
    class TorrentItem(Item):
        url = Field()
        name = Field()
        description = Field()
        size = Field()
###写一个提取数据的爬虫###

接下来需要写一个Spider来定义开始URL，从页面获取url的规则和获取数据的规则。

如果看一下每个torrent的页面内容，就会发现所有的链接都是形如：`http://www.mininova.org/tor/NUMBER`
（NUMBER是一个数据）这样的格式，这样可以用正则表达式来提取连接：/tor/\d+.

我们采用XPath从HTML页面选择要提取的数据，可以看一下其中的一个页面：[http://www.mininova.org/tor/2657665](http://www.mininova.org/tor/2657665)

看了页面源码之后，可以用XPath选择我们需要的数据： torrent的名字、描述和大小。

不难发现torrent的名字包含在`<h1>`标签中：

    <h1>Home[2009][Eng]XviD-ovd</h1>
获取名字的XPath表达式为：

    //h1/text()
描述包含在带有id=”description”的<div>标签中：

    <div id="description">
    "HOME" - a documentary film by Yann Arthus-Bertrand
    <br/>
    <br/>
    ***
    <br/>
    <br/>
    "We are living in exceptional times. Scientists tell us that we have 10 years to change the way we live, avert the depletion of natural resources and the catastrophic evolution of the Earth's climate.
 
    ...
获取描述的XPath是这样的：

    //div[@id='description']
如果想知道更多关于XPath的信息，请看XPath参考。

最后，这是我们的Spider代码：

    class MininovaSpider(CrawlSpider):
 
        name = 'mininova.org'
        allowed_domains = ['mininova.org']
        start_urls = ['http://www.mininova.org/today']
        rules = [Rule(SgmlLinkExtractor(allow=['/tor/\d+']), 'parse_torrent')]
 
        def parse_torrent(self, response):
            x = HtmlXPathSelector(response)
 
            torrent = TorrentItem()
            torrent['url'] = response.url
            torrent['name'] = x.select("//h1/text()").extract()
            torrent['description'] = x.select("//div[@id='description']").extract()
            torrent['size'] = x.select("//div[@id='info-left']/p[2]/text()[2]").extract()
            return torrent
###运行Spider提取数据###

最后我们运行爬虫程序将提取出来的数据以JSON格式写入scraped_data.json文件中。

scrapy crawl mininova.org -o scraped_data.json -t json
这个是用feed export创建JSON文件，你可以很方便的变换格式（例如XML或者CVS）或者存到别的地方（FTP或者Amazon S3）。

也可以用管道（piplines）将数据存入数据库中。

###预览数据###

当获取数据之后你看到scraped_data.json文件中的数据是这样的：

    {"url": "http://www.mininova.org/tor/2657665", "name": ["Home[2009][Eng]XviD-ovd"], "description": ["HOME - a documentary film by ..."], "size": ["699.69 megabyte"]},
    # ... other items ...
    ]
你可以看到所有字段值（除了直接复制的url）职位都是list，这是因为选择器（selectors）返回的是list。你可能想要存储单独的值，或者执行额外的解析/清理值，Item Loaders可以做到。
