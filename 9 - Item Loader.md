##Item Loader

####使用Item Loaders to populate items

要使用Item Loader, 你必须先将它实例化.你可以使用类似字典的对象(eg. Item or dict)来进行实例化,或者不使用对象也可以起.当不用对象进行实例化的时候,Item会自动使用ItemLoader.default_item_class 属性中制定的Item泪在Item Loader constructor中实例化.


然后,你开始收集数值到Item Loader时,通常使用XPath Selector.您可以在同一个item field 里面添加多个数值;Item Loader将知道如何用合适的处理函数来“添加”这些数值.

下面是一个Spider中典型的Item Loader用法,使用Item Chapter中说明的Product Item:

	from scrapy.contrib.loader import XPathItemLoader
	from myproject.items import Product

	def parse(self, response):
    	l = XPathItemLoader(item=Product(), response=response)
    	l.add_xpath('name', '//div[@class="product_name"]')
    	l.add_xpath('name', '//div[@class="product_title"]')
    	l.add_xpath('price', '//p[@id="price"]')
    	l.add_xpath('stock', '//p[@id="stock"]')
    	l.add_value('last_updated', 'today') # you can also use literal values
    	return l.load_item()

快速查看这些代码之后,我们可以看到发现name字段被从页面中两个不同的XPath位置提取:

	1.//div[@class="product_name"]
	2.//div[@class="product_title"]
	
换言之,数据通过用add_xpath()方法,把从两个不同的XPath位置提取的数据收集起来,这是将在以后分配给name字段中的数据｡

之后,类似的请求被用于price和stock字段,最后last_update字段直接不同的方法add_value()添加值(today).

最后,当所有数据被收集时, 调用ItemLoader.load_item ()方法时,实际上填充并返回了填充了先前调用add_xpath ()和add_value ()所收集的数据的Item｡

##输入和输出处理器

Item Loader在每个Item中都包含了一个输入处理器和一个输出处理器｡输入处理器收到数据时(通过add_xpath ()或add_value ()方法),之后输入处理器的结果被收集起来并且保存在ItemLoader内｡收集到所有的数据后, 调用ItemLoader.load_item ( )方法来填充,并得到填充后的Item对象｡这是当输出处理器被和之前收集到的数据(和用输入处理器处理的)被调用.输出处理器的结果是被分配到Item的最终值｡

让我们看一个例子来说明如何输入和输出处理器被一个特定的字段调用(同样适用于其他领域):

	L = XPathItemLoader (Product( ) , some_xpath_selector )
	l.add_xpath ( 'name' , xpath1 ) # ( 1 )
	l.add_xpath ( 'name' , xpath2 ) # ( 2 )
	l.add_value ( 'name' , 'test' ) # ( 3 )
	return l.load_item ( ) # ( 4 )

发生了这些事情:

1. 从xpath1提取出的数据,传递给输入处理器的name字段.输入处理器的结果被收集和保存在Item Loader中(但尚未分配给该Item)｡
2. 从xpath2提取出来的数据,传递给(1)中使用的相同的输入处理器.输入处理器的结果被附加到在(1)中收集的数据(如果有的话) ｡
3. 这种情况下,与之前的相类似,但是不同的是要收集的值被直接分配,而不是从一个XPath的被提取｡但是,该值仍然可以通过输入处理器通过｡在这种情况下,由于该值是不是可迭代它被转换为一个单一的元件的迭代将它传递到输入处理器之前,由于输入处理器总是得到迭代对象｡
4. 收集的数据(1)和(2)传给输出处理器的name字段的｡输出处理器的结果是在Item分配给name字段的值｡

这是值得注意的是,处理器只是可调用的对象,这是所谓的数据进行解析,并返回解析后的值｡所以,你可以使用任何功能为输入或输出处理器｡唯一的要求是,他们必须接受一个(也是唯一一个)位置参数,这将是一个迭代器｡

你需要记住的另一件事是,通过输入处理器返回的值在内部收集(在列表中) ,然后传递到输出处理器来填充字段｡

最后,但并非不重要, Scrapy附带了一些内置的方便常用的处理器｡

##声明输入和输出处理器

正如前面一节中,输入和输出处理器可以在Item Loader定义中声明,这是非常常见的用这种方式来声明输入处理器｡然而,还有一个地方,你可以指定输入和输出的处理器使用方法:Item Field metadata｡下面是一个例子:

	from scrapy.item import Item, Field
	from scrapy.contrib.loader.processor import MapCompose, Join, TakeFirst

	from scrapy.utils.markup import remove_entities
	from myproject.utils import filter_prices

	class Product(Item):
    	name = Field(
        	input_processor=MapCompose(remove_entities),
        	output_processor=Join(),
    	)
    	price = Field(
        	default=0,
        	input_processor=MapCompose(remove_entities, filter_prices),
        	output_processor=TakeFirst(),
    	)
    	
优先级顺序,用于输入和输出的处理器,如下所示:

1. Item Loader 字段特有的属性: field_in和field_out (最优先)
2. 字段元数据( input_processor和output_processor的键值)
3. Item Loader默认值: ItemLoader.default_input_processor ( )和ItemLoader.default_output_processor ( ) (最低优先级)

另请参阅:Reusing and extending Item Loaders.

##Item Loader Context

该Item Loader Context是在Item Loader中的所有输入和输出处理器之间拥有任意的键/值的共享字典｡它可以声明,实例化和使用Item Loader时,可以通过｡它们被用来修改输入/输出处理器的行为｡

例如,假设你有一个函数parse_length接收文本值,并提取它的长度:

	def parse_length(text, loader_context):
    	unit = loader_context.get('unit', 'm')
    	# ... length parsing code goes here ...
    	return parsed_length
    
通过接受loader_context参数的函数被明确告诉Item Loader 即能获得该Item Loader Context,因此调用时,它的Item Loader通过当前活动的情况下,处理器的函数(在这种情况下是parse_length),因此可以使用它们｡

有几种方法来修改Item Loader Context值:

1. 通过修改当前活动Item Loader Context(context属性) :

		loader = ItemLoader(product)
		loader.context['unit'] = 'cm'
	
2. 在Item Loader实例化的时候(Item Loader构造的关键字参数都存储在Item Loader Context) :

		loader = ItemLoader(product, unit='cm')

3. 在Item Loader声明,对于那些支持与Item Loader Context 实例化的输入/输出处理器｡ MapCompose就是其中之一:

		class ProductLoader(ItemLoader):
    		length_out = MapCompose(parse_length, unit='cm')

##Item Loader 对象

	class scrapy.contrib.loader.ItemLoader([item], **kwargs)

返回一个新的Item Loader用于填充给定的Item｡如果没有Item被给予,一种是使用类中default_item_class自动实例化｡

该Item和其余关键字参数分配给Loader的context(通过context属性访问).

	get_value(value, *processors, **kwargs)

由给定的处理器和关键字参数处理给定value｡

可用关键字参数:

示例:

	>>> from scrapy.contrib.loader.processor import TakeFirst
	>>> loader.get_value(u'name: foo', TakeFirst(), unicode.upper, re='name: (.+)')
	'FOO`
	
-

	add_value(field_name, value, *processors, **kwargs)
处理后把给定value添加给定域｡

该值首先通过的get_value ()使用指定的处理器和kwargs参数 ,然后通过该字段的field input processor传递,其结果附加给为该字段收集的数据｡如果字段已经包含收集到的数据,新的数据被添加｡

给定FIELD_NAME可以为None,其中可加入多个字段的情况下的值｡并将处理后的值应与FIELD_NAME一个字典映射到值｡

示例:

	loader.add_value('name', u'Color TV')
	loader.add_value('colours', [u'white', u'blue'])
	loader.add_value('length', u'100')
	loader.add_value('name', u'name: foo', TakeFirst(), re='name: (.+)')
	loader.add_value(None, {'name': u'foo', 'sex': u'male'})

-

	replace_value(field_name, value)
以add_value相似( ),但仅仅把收集到的值替换为新值,而非添加｡

	load_item ( )
填充与项目到目前为止收集到的数据,并返回它｡收集到的数据首先通过输出处理器传递的最终的值分配给每个Item Field｡

	get_collected_values ​​( FIELD_NAME )
返回给定field收集的值｡

	get_output_value ( FIELD_NAME )
返回给定field中通过输出处理器解析的收集数据中.此方法不用于填充或修改项目的｡

	get_input_processor ( FIELD_NAME )
返回给定field的输入处理器｡

	get_output_processor ( FIELD_NAME )
返回给定field的输出处理器｡

	Item
Item对象被Item Loader解析｡

	Context
该Item Loader的当前活动Context｡

	default_item_class
Item类,用于在构造函数中没有给出情况下实例化items｡

	default_input_processor
默认的输入处理器用于没有指定处理器的field｡

	default_output_processor
默认的输出处理器用于没有指定处理器的field｡

	class scrapy.contrib.loader.XPathItemLoader([item, selector, response], **kwargs)
该XPathItemLoader类是ItemLoader类的扩展,它提供更方便的机制,使用XPath Selector来提取网页数据｡

XPathItemLoader对象可以在它们的构造函数是多接受两个附加参数:

	get_xpath(xpath, *processors, **kwargs)
类似ItemLoader.get_value (),但接收到是XPath,而不是一个值,该值被用于提取的Unicode字符串列表从与此XPathItemLoader相关联的Selector｡

示例:

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.get_xpath('//p[@class="product-name"]')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.get_xpath('//p[@id="price"]', TakeFirst(), re='the price is (.*)')
 
 -

	add_xpath(field_name, xpath, *processors, **kwargs)		
类似ItemLoader.add_value (),但接收到是XPath,而不是一个值,该值被用于提取的Unicode字符串列表从与此XPathItemLoader相关联的Selector｡

见get_xpath() for kwargs ｡

示例:

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.add_xpath('name', '//p[@class="product-name"]')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.add_xpath('price', '//p[@id="price"]', re='the price is (.*)')
	loader.add_xpath ('价格',' / / P [ @ ID = “价格”] “ ,再='价格( * ) ')

-
	
	replace_xpath(field_name, xpath, *processors, **kwargs)		
类似add_xpath (),但替换收集的数据,而不是添加｡

default_selector_class
这个类用于构造此XPathItemLoader的选择,如果只有一个响应中给出了构造函数｡如果选择是由于在构造函数中这个属性被忽略｡此属性在子类中有时覆盖｡

	Selector
该XPathSelector对象从提取数据｡它无论是在构造函数或一个来自使用default_selector_class在构造函数中给出的响应创建给定的选择器｡此属性是只读｡

##重用和扩展Item Loader

随着项目的逐渐变大,并获得越来越多的Spider,维护成为一个根本性的问题,特别是当你要处理的每个Spider许多不同的解析规则,有很多exception,也想重用那些通用处理器｡

Item Loader旨在缓解解析规则的维护负担,却又不失灵活性,并在同一时间,扩展和重写规则提供了方便的机制.为此Item Loader支持传统的Python类的继承来处理特定的Spider(或蜘蛛组)的差异｡

举个例子,一些特定的网站用三个破折号围住他们的产品名称(即---等离子电视--- ) ,你不希望最终爬取那些破折号在最终产品的名称中｡

这里,你可以通过重用和扩展默认Product Item Loader (ProductLoader)来删除那些破折号:

	from scrapy.contrib.loader.processor import MapCompose
	from myproject.ItemLoaders import ProductLoader

	def strip_dashes(x):
    	return x.strip('-')

	class SiteSpecificLoader(ProductLoader):
    	name_in = MapCompose(strip_dashes, ProductLoader.name_in)
    	
另一种情况,拓展Item Loader是非常有帮助的,当你有多个源格式,例如XML和HTML ｡在XML版本,你可能希望删除CDATA事件｡以下是如何做到这一点的例子:

	from scrapy.contrib.loader.processor import MapCompose
	from myproject.ItemLoaders import ProductLoader
	from myproject.utils.xml import remove_cdata

	class XmlProductLoader(ProductLoader):
    	name_in = MapCompose(remove_cdata, ProductLoader.name_in)

这就是你如何典型地extend输入处理器｡

至于输出处理器,它是比较常见在该field metadata声明他们,因为它们通常只依赖于field,而不是在每个特定网站的解析规则(如输入处理器那样) ｡另请参阅: Declaring Input and Output Processors｡

还有许多其他可能的方式来扩展,继承并重写你的Item Loader,以及不同的Item Loader层次结构可能更适合为不同的项目｡ Scrapy只提供机制,它并不会给你Loader集合的任何特定结构 - 这是由你和你的项目求来决定｡

##可用的内置处理器

虽然你可以使用任何可调用的函数作为输入和输出的处理器, Scrapy提供了一些常用的处理器,如下所述｡其中有些,像MapCompose (这通常被用作输入处理器)组成的顺序执行多种功能的输出,以产生最终的分析值｡

这里是所有内置的处理器列表:

	class scrapy.contrib.loader.processor.Identity
最简单的处理器,并不做任何事情｡它返回原来的值｡它不接受任何参数也不接受Loader Context｡

例如:

	>>> from scrapy.contrib.loader.processor import Identity
	>>> proc = Identity()
	>>> proc(['one', 'two', 'three'])
	['one', 'two', 'three']
	
-

	class scrapy.contrib.loader.processor.TakeFirst
	
返回从接收到值中第一个non-null/non-empty值,所以它通常被用作一个输出处理器为单值字段｡它不接受任何构造函数的参数,也不接受Loader Context｡

例如:

	>>> from scrapy.contrib.loader.processor import TakeFirst
	>>> proc = TakeFirst()
	>>> proc(['', 'one', 'two', 'three'])
	'one'
	
-
	
	class scrapy.contrib.loader.processor.Join(separator=u' ')
返回加入了在构造函数中给定的分隔符的值,默认为u' '｡它不接受Loader Context｡

当使用默认的分隔符,这个处理器相当于函数:u' '.join｡

示例

	>>> from scrapy.contrib.loader.processor import Join
	>>> proc = Join()
	>>> proc(['one', 'two', 'three'])
	u'one two three'
	>>> proc = Join('<br>')
	>>> proc(['one', 'two', 'three'])
	u'one<br>two<br>three
	
-
	
	class scrapy.contrib.loader.processor.Compose(*functions, **default_loader_context)
	
这是由给定函数构成的处理器｡这意味着,该处理器的每个输入值被传递到第一个函数,并且该函数的结果被传递给第二个函数,等等,直到最后一个函数返回该处理器的输出值｡

默认情况下,停在None值｡这种行为可以通过关键字参数stop_on_none = false来改变｡

例如:

	>>> from scrapy.contrib.loader.processor import Compose
	>>> proc = Compose(lambda v: v[0], str.upper)
	>>> proc(['hello', 'world'])
	'HELLO'

每个功能可以选择性地接收loader_context参数｡对于那些做什么,这款处理器将通过当前活动的Loader context通过该参数｡

在构造函数中传递的关键字参数被用作传递给每个函数调用的默认Loader context值｡然而,传递给函数的最终Loader context值与通过ItemLoader.context ( )属性访问当前活动的Loader context覆盖｡
	
	class scrapy.contrib.loader.processor.MapCompose(*functions, **default_loader_context)
	
这是由给定的函数,类似于在compose processor,构成的处理器｡采用该处理器所不同的是内部结果在函数间传递的方式,具体如下所示:

此处理器的输入值被迭代并且每个元素被传递到第一个函数,并且该函数(每个元素)的结果被连接起来以构造一个新的迭代,然后将其传递到第二个函数,等等,直到最后一个函数应用迄今收集的值的列表中的每个值｡最后一个函数的输出值串联在一起,以产生这种处理器的输出｡

每一个特定的函数可以返回一个或多个值,施加到另一输入端的值相同的函数返回的值的列表中的一个列表｡该函数也可以返回None,在这种情况下,该函数的输出被忽略,以便进一步处理｡

该处理器提供了一个方便的方法来编写只对只与单个值(而不是可迭代对象)有效的函数由于这个原因, MapCompose处理器通常用作输入处理器中,由于数据是通过Selectors 的 extact()方法,该方法返回Unicode字符串的列表｡

下面的例子应该说明它是如何工作的:

	>>> def filter_world(x):
	...     return None if x == 'world' else x
	...
	>>> from scrapy.contrib.loader.processor import MapCompose
	>>> proc = MapCompose(filter_world, unicode.upper)
	>>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
	[u'HELLO, u'THIS', u'IS', u'SCRAPY']
	
由于Compose Loader,函数可以接收Loader Context和构造关键字参数作为默认的context值｡见Compose Processor的详细信息｡