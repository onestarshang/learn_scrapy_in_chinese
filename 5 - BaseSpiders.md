#Spiders

spiders是一组类，用来定义怎么爬取某个或者某些站点，包括如何展现爬取行为的、如何从页面中提取结构化数据的。换言之，spider是一个自定义爬取行为和解析某个或者某些特定的页面的地方。

对于Spider类来说，它的抓取流程是这样的：

你从产生初始化第一个链接URLs请求开始，然后定义一个可以传递从这

- 个请求返回的Response的回调函数。第一次执行的Requests是从start_requests() 方法中获取的，这个方法中的Requests是在start_urls变量中定义的；而parse方法作为这些Requests回调函数。
- 在回调函数中，你解析response（web页面），然后返回items对象、Requests对象或者这两者的迭代器。这些Requests也许也包含回调函数（可能相同）然后通过Spider处理（download），最后这些response被指定的回调函数处理。
- 在回调函数中，一般情况下会用Selector类处理页面内容（当然也可以用BeautifulSoup、lxml或者你喜欢的任何功能），然后产生出带有解析数据的items。
- 最后，从spider中返回的items会被不断的存入数据表中（一些Item Pipeline），或者利用Feed exports写入文件。

尽管这个流程适用于任何Spider类，但是Scrapy中的不同Spider的具体功能不尽相同，我们将一一介绍。

##Spider 参数

Spider类可以接受改变他们行为的变量。通常的用法就是定义起始的URLs或者对站点的某个部分做限制，但是不能用作spider的配置功能。

spider类的参数通过crawl命令行传入：

scrapy crawl myspider -a category=electronics
spider通过构造函数接收参数：

    class MySpider(BaseSpider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
        super(MySpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]
        # ...

spider的参数也可以通过Scrapyd的schedule.json传递。

##内置Spider类参考

Scrapy自带一些通用的spider类，你可以继承它们然后使用。目的在于给一些常用的爬站案例提供简单的功能特性。比如根据某个规则获取所有链接，爬取Sitemaps或者解析XML/CSV。

在以下用到的spiders类中，我们都默认你已经配置好你的items：

    from scrapy.item import Item
 
    class TestItem(Item):
        id = Field()
        name = Field()
        description = Field()

##BaseSpider

 *class scrapy.spider.BaseSpider*

这是最简单的一个spider类，其他类都继承自这个类，包括Scrapy中的其他spider类还是自定义的的spider类。它没有提供任何其他的功能特性，只需要给出start_urls/start_requests，调用parse方法处理这些response。

**name**

定义了这个spider的名字的字符串，这个名字的作用是Scrapy如何定位spider 的，所以必须是唯一的。因为你不能获得更多信息如果实例化多个相同的spider类，这是最重要的spider类的属性并且是必须的。

如果爬取一个特定的网址，通常情况下这个spider类用这个网址的名称命名，不带TDL。例如：你要爬取的站名是mywebsite.com，则spider类的名字为mywebsite。

**allowed_domains**

一个可选的字符串列表，包含可以爬取的站点链接。如果请求链接不在这个列表中并且OffsiteMiddleware被设置为enable，就不会被执行。

**start_urls**

如果没有指定特定的请求链接，那么spider类就从这个list中定义的链接开始执行。所以最初下载下来的页面请求会被列在这里。随后的其他的链接会从start URLs请求的页面包含的数据中产生。

**start_requests()**

这个方法必须返回spider类中的第一个Requests的迭代器。

Scrapy会调用这个方法，当一个spider被打开但是没有特别指定URLs的时候。如果特定的URLs被指定，就会调用make_requests_from_url()生成Requests。Scrapy只会调用这个方法一次，所以它作为一个生成器来实现是很安全的。

默认从start_urls中生成Requests的方法是make_requests_from_url()。

如果你想改变用作开始地址的Requests，可以重载（override）这个方法。例如：你想通过POST方法作为起始请求，你可以这样：

    def start_requests(self):
        return [FormRequest("http://www.example.com/login",
                formdata={'user': 'john', 'pass': 'secret'},
                callback=self.logged_in)]
    def logged_in(self, response):
        # here you would extract links to follow and return Requests for
        # each of them, with another callback
        pass
**make_requests_from_url(url)**

这个方法接受一个URL并且返回要爬取的Request对象，这个方法通常在start_requests()方法中构造初始请求，一般是用来改变url请求的。

除非被重载，这个方法可以返回将parse视为回调函数的Requests，并且dont_filter是被允许的。

**parse(response)**

当没特别指定一个回调函数的时候，Scrapy默认调用这个回调函数处理返回的response。

parse方法负责处理response，返回数据或者其他的URLs。而这些其他的Requests的回调函数的需求和BaseSpider类一样。

这个被其他任何Requests调用的方法返回Request或者Item对象迭代器。

Parameters: 

- response (:class:~scrapy.http.Response`) – 解析的response

- log(message[, level, component])

用scrapy.log.msg()方法记录日志信息，用spider的名字自动填充spider的参数。

##BaseSpider示例

例子：

    from scrapy import log # This module is useful for printing out debug information
    from scrapy.spider import BaseSpider
 
    class MySpider(BaseSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
              'http://www.example.com/1.html',
              'http://www.example.com/2.html',
              'http://www.example.com/3.html',
              ]
 
        def parse(self, response):
            self.log('A response from %s just arrived!' % response.url)
另外一个从一个调用中返回多个Requests和Item对象的例子：

    from scrapy.selector import HtmlXPathSelector
    from scrapy.spider import BaseSpider
    from scrapy.http import Request
    from myproject.items import MyItem
 
    class MySpider(BaseSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
             'http://www.example.com/1.html',
             'http://www.example.com/2.html',
             'http://www.example.com/3.html',
             ]
 
        def parse(self, response):
            hxs = HtmlXPathSelector(response)
            for h3 in hxs.select('//h3').extract():
                yield MyItem(title=h3)
 
            for url in hxs.select('//a/@href').extract():
                yield Request(url, callback=self.parse)
 
