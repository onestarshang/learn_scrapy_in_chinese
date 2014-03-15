#Selectors
当你爬取网页时，你最常做的事情就是从HTML网页中抓取数据。这里有一些可用的库帮助你完成这件事情：
 - BeautyfulSoup在python程序员中非常流行的一个库，它可以将HTML页面源码转化为对象，并且可以很好的处理较差的HTML语言标记，它的缺点是：太慢。
 - lxml是根据python的ElementTree的API实现的XML解析（也可以进行HTML解析）的库。

而Scrapy是使用的自己提取数据的机制。称它们为selector是因为只需要利用它根据XPath或者CSS表达式规则获取HTML文档中的部分内容。

XPath是一门语言，既可以选取XML文档中的节点，也可以选取HTML文档中的节点。CSS是一门语言，它的作用是为HTML文档添加样式。它定义了和这些样式相关的HTML元素的selector。

Scrapy的selector是以lxml为基础的，这意味它在数据和解析上和lxml非常接近。

这部分解释了selector是怎样工作的和它小巧简单的API，它不像lxml那样，lxml除了选取标记语言式文档中的节点，还有更多的API做很多其他的工作。

看Selector完整API请移步[Selector reference](http://scrapy.readthedocs.org/en/latest/topics/selectors.html#topics-selectors-ref)。

##使用Selectors
###创建 selectors
将Response当做第一个参数传入<code>Selector</code>类中，就能将Selector实例化。而Response的body就是select提取数据的来源。
<pre><code>from scrapy.spider import Spider
from scrapy.selector import Selector

class MySpider(Spider):
    # ...
    def parse(self, response):
        sel = Selector(response)
        # Using XPath query
        print sel.xpath('//p')
        # Using CSS query
        print sel.css('p')
        # Nesting queries
        print sel.xpath('//div[@foo="bar"]').css('span#bold')</code></pre>
        

###使用selectors
我们将使用Scrapy 的shell命令和一个已经存在于Scrapy文件服务器上的文档作为例子，解释如何使用selectors。

这里是HTML源码：

<code>...</code>

首先，打开shell
<pre><code>scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html</code></pre>

当shell加载之后，会自动初始化一个名为<code>sel</code>的selector对象。由于我们要解析HTML，所以<code>sel</code>默认使用HTML的parser。

根据上面的HTML代码，我们来生成一个提取title标签中文本的XPath。

    >>> sel.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]</code>

如你所见，<code> .xpath()</code>返回一个<code>SelectorList</code>实例，它是一个列出了新的selectors的列表（list）。这个API可以很快的选取嵌套的数据。

要想真正的提取文本信息，你必须调用selector的<code>.extract()</code>方法：
    
    >>> sel.xpath('//title/text()').extract()
    [u'Example website']</code>

值得注意的是CSS的selector用的是CSS3 pseudo-elements提取文本或者属性节点：
    
    >>> sel.css('title::text').extract()
    [u'Example website']

现在我们来获取一下常规的URL和一些图片的链接：
    
    >>> sel.xpath('//base/@href').extract()
    [u'http://example.com/']
    
    >>> sel.css('base::attr(href)').extract()
    [u'http://example.com/']
    
    >>> sel.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']
     
    >>> sel.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']
     
    >>> sel.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']
     
    >>> sel.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']
 
 
 ###嵌套（遍历）selectors
 selector的提取节点的方法（<code>.xpath()</code>or <code>.css()</code>）返回的是相同selector类型的列表，所以你也可以调用他们的提取节点的方法。这里有个例子：
    
    >>> links = sel.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
     u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
     u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
     u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
     u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']
     
    >>> for index, link in enumerate(links):
        args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
        print 'Link number %d points to url %s and image %s' % args
        
    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']

###在selector中利用正则表达式

Selector中也有可以利用正则表达式提取数据的方法<code>.re()</code>，但是和<code>.xpath()</code>或者<code>.css</code>不同的是，<code>.re()</code>返回的是unicode字符串。所以你不能迭代的调用<code>.re()</code>方法。
    >>> sel.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']
 
 
###使用XPths()的相对路径的方法

一定要记住，如果你要遍历的使用selectors或者XPath的表达式用"/"开头的，那么XPath表示的是文档中节点的据对路径，而不是相对路径。

例如，如果你想要获取<code>`<div>`</code>标签下的所有<code>`<p>`</code>节点，首先你应该获取所有的`<div>`节点：
    
    >>> divs = sel.xpath('//div')

一开始，你可能会尝试下面这种错误的写法，它实际上不是获取`<div>`中的`<p>`节点，而是文档中所有的`<p>`节点：
    
    >>> for p in divs.xpath('//p')  # this is wrong - gets all <p> from the whole document
    >>>     print p.extract()

有一种优雅的方法是：
    
    >>> for p in divs.xpath('.//p')  # extracts all <p> inside
    >>>     print p.extract()

另外一种方法是获取所有直属的`<p>`节点：
    
    >>> for p in divs.xpath('p') 
    >>>     print p.extract()

更多细节请移步[ Location Paths](http://www.w3.org/TR/xpath/#location-paths)部分


###使用EXSL拓展

