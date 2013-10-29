###CrawlSpider

*class scrapy.contrib.spiders.CrawlSpider*

这是抓取一般网页最常用的类，它提供了一种简单的机制，能够为爬取的链接定义一组规则。这个类也许不是一些特定网站或项目的最优选择，但是能够满足一般通用的情况，所以你可以从它开始，或者根据自己的需求重写，或者直接实现你自己的爬虫类。

**rules**

这是一个Rule对象列表，每条规则定义了爬取网站链接的行为，Rules对象在下面定义，如果一条连接命中多条规则，以第一条规则进行匹配，顺序由属性中定义的顺序决定。

这个类也提供一个可重载的方法：

**parse_start_url(response)**

这个方法被start_urls的response调用，它可以解析初始的response，并且必须返回item对象或者Request对象，或者一个包含两者中任意一个的迭代器。


###Crawling rules

*class scrapy.contrib.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)*

link_extractor是一个Link Extractor对象，定义怎样提取每个需要爬取的页面中的链接。

callback是一个可调用方法或者一个字符串（spider类中用这个字符串命名的方法）会被每个指定的link_extractor调用，这个方法的第一个参数是response必须返回一个item或者Request的list。

cb_kwargs是一个包含关键字参数的字典，可以传递给callback函数。

follow是一个布尔值，指定这些通过规则匹配出来的链接是否需要继续，如果callback是None，follow默认为False，否则follow是True。

process_links 是一个可调用方法或者一个字符串（spider类中用这个字符串命名的方法）会被每个指定的link_extractor调用，这个主要作用是过滤。

process_request是一个可调用方法或者一个字符串（spider类中用这个字符串命名的方法）会被这个规则的每个request调用，必须返回一个request或者None。

###CrawlSpider 示例

我们来看一个示例：

    from scrapy.contrib.spiders import CrawlSpider, Rule
    from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
    from scrapy.selector import Selector
    from scrapy.item import Item
 
    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']
 
        rules = (
        # Extract links matching 'category.php' (but not matching 'subsection.php')
        # and follow links from them (since no callback means follow=True by default).
        Rule(SgmlLinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),
 
        # Extract links matching 'item.php' and parse them with the spider's method parse_item
        Rule(SgmlLinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )
 
        def parse_item(self, response):
            self.log('Hi, this is an item page! %s' % response.url)
 
            sel = Selector(response)
            item = Item()
            item['id'] = sel.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = sel.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = sel.xpath('//td[@id="item_description"]/text()').extract()
            return item
这个spider类从example.com的home页开始，收集category链接、item链接，并解利用parse_item方法析后者，每个item的response，都会有数据用XPath解析出来，值存入item对象中。
