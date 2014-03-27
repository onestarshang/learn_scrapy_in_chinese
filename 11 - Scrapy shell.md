#Scrapy shell#

scrapy shell 是一种交互式shell，你可以在上面快速的尝试和调试你的爬虫代码，而不必去运行爬虫。那意味着对于测试数据提取是有帮助的，但是事实上你可以测试任何种类的代码，只要它也是常规的Python shell.

这种shell被用来测试XPath或者CSS表达式，可以它们是怎么工作的，是怎么样从你正在抓取的页面上提取数据的。当你正在写你的爬虫时，它也允许你交互式的测试你的表达式，而不必去运行你的爬虫来测试每个变化。

一旦你熟悉了Scrapy shell，你将会看到，对于开发和调试你的爬虫，是一种很有价值的工具。

如果你已经安装了[IPython](http://ipython.org/)，Scrapy shell将会使用IPython，而不是标准的Python console,[IPython](http://ipython.org/)控制台是非常强大的，除了其他事情之外它还提供了智能的自动补全和有颜色的输出。

我们强烈建议安装IPython,特别的当你的工作是在IPython擅长的Unix系统上，更多信息，请看[IPython installation guide](http://ipython.org/install.html)。

###启动shell###

为了启动Scrapy shell，你可以使用shell命令，像这样：

    crapy shell <url>

<url>是你要抓取的URL。

###使用shell###

Scrapy shell是一种常规的Python控制台（或者是IPython控制台，如果它可用），它可以提供额外的方便的快捷函数。

###可用的快捷函数###

- shelp() - 打印一系列可用的快捷函数和对象的帮助信息
- fetch(request_or_url) - 从给定的请求或者URL中获取一个新的response，并且有根据的更新所有相关的对象。
- view(response) - 在你本地的web浏览器中打开一个给定的response来进行检查。这将会在response的body上增加一个[base](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base)标签，这样外部的链接（例如images和样式表）可以正常的显示。注意，那将会在你的计算机中创建一个临时文件，这个临时文件将不会自动移除。

###可用的Scrapy对象###

Scrapy shell将会根据下载页面自动创建一些便捷的对象，像[Response](http://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象和[Selector](http://doc.scrapy.org/en/latest/topics/selectors.html#scrapy.selector.Selector)对象（对于HTML和XML内容来说）。

这些对象是:

- crawler - 当前的[Crawler](http://doc.scrapy.org/en/latest/topics/api.html#scrapy.crawler.Crawler)对象。
- spider - 处理URL的爬虫，如果在当前URL中没找到爬虫，那么就是一个[Spider](http://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spider.Spider)对象。
- request - 上一个获取的页面的[Request](http://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象。你可用通过[replace()](http://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.replace)方法来修改这个请求或者是通过fetch快捷函数来获取一个新的request对象而不用脱离shell。
- response - 一个包含了上一个获取的页面的[Response](http://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象。
- sel - 一个用上一个获取页面构造的[Selector](http://doc.scrapy.org/en/latest/topics/selectors.html#scrapy.selector.Selector)对象。
- settings - 当前的[Scrapy settings](http://doc.scrapy.org/en/latest/topics/settings.html#topics-settings)。

###shell会话的例子###

这是一个典型的shell会话例子，我们可以通过抓取[http://scrapy.org](http://scrapy.org/)页面，然后开始抓取[http://slashdot.org](http://slashdot.org/)页面，最后，我们把（Slashdot）的请求方法修改为POST方式，并且重新获取，得到一个HTTP 405（请求方式不被允许）的错误。我们通过键入Ctrl-D（在Unix系统）或者Ctrl-C（在Windows系统）结束会话。

记住，当你尝试数据提取的时候可能会不一样，因为这些页面不是静态页面，而且可以根据你的测试随时变化。这个例子的目的仅仅是让你熟悉Scrapy shell的工作原理。

首先，我们启动shell：

    scrapy shell 'http://scrapy.org' --nolog

然后，shell获取URL（通过Scrapy downloader），并且打印出一系列可用的对象和有用的快捷函数（你将会注意到这些行都以[s]前缀开始）。

    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    [s]   item       {}
    [s]   request    <GET http://scrapy.org>
    [s]   response   <200 http://scrapy.org>
    [s]   sel        <Selector xpath=None data=u'<html>\n  <head>\n    <meta charset="utf-8'>
    [s]   settings   <CrawlerSettings module=None>
    [s]   spider     <Spider 'default' at 0x20c6f50>
    [s] Useful shortcuts:
    [s]   shelp()           Shell help (print this help)
    [s]   fetch(req_or_url) Fetch request (or URL) and update local objects
    [s]   view(response)    View response in a browser

    >>>

然后，我们可以使用这些对象：

    >>> sel.xpath("//h2/text()").extract()[0]
    u'Welcome to Scrapy'

    >>> fetch("http://slashdot.org")
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1a13b50>
    [s]   item       {}
    [s]   request    <GET http://slashdot.org>
    [s]   response   <200 http://slashdot.org>
    [s]   sel        <Selector xpath=None data=u'<html lang="en">\n<head>\n\n\n\n\n<script id="'>
    [s]   settings   <CrawlerSettings module=None>
    [s]   spider     <Spider 'default' at 0x20c6f50>
    [s] Useful shortcuts:
    [s]   shelp()           Shell help (print this help)
    [s]   fetch(req_or_url) Fetch request (or URL) and update local objects
    [s]   view(response)    View response in a browser

    >>> sel.xpath('//title/text()').extract()
    [u'Slashdot: News for nerds, stuff that matters']

    >>> request = request.replace(method="POST")

    >>> fetch(request)
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    ...

    >>>

###从爬虫来调用shell来检查响应###

有时你想要在你的爬虫中的一个特定的点来检查正在被处理的响应，看你所期望的响应是否到那.

这个可以通过`scrapy.shell.inspect_response`函数来实现。

这个例子展示了你将怎样从你的爬虫中来调用它。

    from scrapy.spider import Spider


    class MySpider(Spider):
        name = "myspider"
        start_urls = [
            "http://example.com",
            "http://example.org",
            "http://example.net",
        ]

        def parse(self, response):
            # We want to inspect one specific response.
            if ".org" in response.url:
                from scrapy.shell import inspect_response
                inspect_response(response)

            # Rest of parsing code.

当你运行爬虫的时候，你将会得到类似下面的信息：

    2014-01-23 17:48:31-0400 [myspider] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
    2014-01-23 17:48:31-0400 [myspider] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    ...

    >>> response.url
    'http://example.org'

然后你可以检查提取代码是否工作：

    >>> sel.xpath('//h1[@class="fn"]')
    []

它没有起效。因此你可以在你的web浏览器中打开response来看是否是你所期望的响应。

    >>> view(response)
    True

最后，你键入Ctrl-D（或者在Windows中是Ctrl-C）来退出shell并且继续爬取。

    >>> ^D
    2014-01-23 17:50:03-0400 [myspider] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
    ...

注意，你不能在这里用`fetch`快捷函数，因为shell屏蔽了Scrapy引擎。但是，当你退出shell后，爬虫依然会从它停止的地方继续爬取，如上所示。
