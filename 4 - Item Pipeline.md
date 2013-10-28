#Item Pipeline#

当item被spider类抓取之后，它被传递给pipeline，通过pipeline依次调用一些组件来进行处理。

每一个item pipeline都是一个实现了一些简单方法的类。item pipeline接收item，并执行一些动作，也决定了是否继续处理item或者不处理。

item pipelines的一般用法：

- 简化HTML数据
- 校验爬取来的数据
- 处理重复数据
- 将数据存储到数据库

##写自定义的item pipeline##

写自定义的pipeline很简单，就是一个必须实现一下方法的一个python类：

*process_item(item, spider)*

这个方法被每个itempipeline调用，并且必须返回一个Item对象（或者它的任意一个子类）或者抛出一个DropItem的异常。被Drop的item将不会再被pipeline处理。

**Parameters：**

- item（item对象）- 抓取的item
- spider（Basespider对象）- 抓取item的spider

另外，也可以实现其他的方法：

*open_spider(spider)*

当spider打开的时候，被调用

**Parameters：** spider（Basespider对象）- 被打开的spider

*close_spider(spider)*

当spider关闭的时候，被调用

**Parameters：** spider（Basespider对象）- 被关闭的spider

##Item Pipeline示例##

###价格校验并且丢弃价格为空的item###

看一下下面构造的pipeline，它可以调整那些没有VAT（price_excludes_vat 属性）的item的值，并且丢弃没有价格值的item。

<pre><code>from scrapy.exceptions import DropItem
 
class PricePipeline(object):
 
    vat_factor = 1.15
 
    def process_item(self, item, spider):
        if item['price']:
            if item['price_excludes_vat']:
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)</code></pre>
###将items存入JSON文件###

下面的pipeline将所有的item都存入一个item.jl文件中，每一行都是一个json格式的item：

<pre><code>import json
 
class JsonWriterPipeline(object):
 
    def __init__(self):
        self.file = open('items.jl', 'wb')
 
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + "\n"
        self.file.write(line)
        return item</code></pre>
*注意：*

*JsonWriterPipeline 类的目的是介绍如何用编写item pipeline，如果你真的想将数据存入JSON文件应该使用Feed exports*
###过滤重复items###

这是一个过滤重复item的过滤器，并且丢弃已经处理过的item。比如我们定义的item是有唯一的id，但是spider爬取的item有重复的id：

<pre><code>from scrapy import signals
from scrapy.exceptions import DropItem
 
class DuplicatesPipeline(object):
 
    def __init__(self):
        self.ids_seen = set()
 
    def process_item(self, item, spider):
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item</code></pre>
##激活一个Item Pipeline类##

如果想用item pipeline，你必须将自定义的类添加到ITEM_PIPELINES配置中，像下面这样：

<pre><code>ITEM_PIPELINES = {
'myproject.pipeline.PricePipeline': 300,
'myproject.pipeline.JsonWriterPipeline': 800,
}</code></pre>
