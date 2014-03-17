##Link Extractors

Link Extractors 是那些目的仅仅是从网页中抽取最终将会被follow链接的对象 (scrapy.http.Response objects).


Scrapy默认提供2种可用的 Link Extractor,但你通过实现一个简单的接口创建自己定制的Link Extractor来满足需求｡

每个LinkExtractor有唯一的公共方法是extract_links ,它接收一个Response对象,并返回一个链接列表｡Link Extractors,要实例化一次并且extract_links方法会根据不同的response调用多次提取链接｡

Link Extractors在CrawlSpider类(在Scrapy可用)中使用 ,通过一套规则,但你也可以用它在你的Spider,即使你不是从CrawlSpider继承的子类,因为它的目的很简单:提取链接｡

##内置Link Extractor 参考

所有与Scrapy绑定且可用的Link Extractors类在scrapy.contrib.linkextractors模块提供｡

##Sgml Link Extractor
	class scrapy.contrib.linkextractors.sgml.SgmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), tags=('a', 'area'), attrs=('href'), canonicalize=True, unique=True, process_value=None)
	
该Sgml Link Extractor延伸通过提供额外的过滤器底座BaseSgmlLinkExtractor您可以指定要提取的链接,包括正则表达式模式的链接必须匹配被提取｡所有这些过滤器是通过这些构造函数的参数配置:

参数:

1. allow(a regular expression (or list of)) - 必须要匹配这个正则表达式(或正则表达式列表)的URL才会被提取｡如果没有给出(或None) ,它会匹配所有的链接｡

2. deny (a regular expression (or list of))  - 与这个正则表达式(或正则表达式列表)的(绝对) 不匹配的URL必须被排除在外(即不提取)｡它的优先级高于允许的参数｡如果没有给出(或None) ,将不排除任何链接｡

3. allow_domains (str or list) - 单值或者包含字符串域的列表表示会被提取的链接的domains
4. deny_domains (str or list) - 单值或包含域名的字符串,将不考虑提取链接的domains
5. deny_extensions (list) - 应提取链接时,可以忽略扩展名的列表｡如果没有给出,它会默认为scrapy.linkextractor模块中定义的ignored_extensions列表｡
6. restrict_xpaths (str or list) - 一个的XPath (或XPath的列表),它定义了链路应该从提取的响应内的区域｡如果给定的,只有那些XPath的选择的文本将被扫描的链接｡见下面的例子｡
7. tags (str or list)  - 提取链接时要考虑的标记或标记列表｡默认为( 'a' , 'area') ｡
8. attrs (boolean) - 提取链接时应该寻找的attrbitues列表(仅在tag参数中指定的标签)｡默认为('href')
9. canonicalize (boolean) - 规范化每次提取的URL (使用scrapy.utils.url.canonicalize_url )｡默认为True ｡
10. unique (boolean) - 重复过滤是否应适用于提取的链接｡
11. process_value (callable) - 见Base Sgml Link Extractor类的构造函数process_value参数


##Base Sgml Link Extractor
	
		class scrapy.contrib.linkextractors.sgml.BaseSgmlLinkExtractor(tag="a", attr="href", unique=False, process_value=None)
	
这个Link Extractor的目的只是充当了Sgml Link Extractor的基类｡你应该使用Sgml Link Extractor｡

该构造函数的参数是:

参数:

1. tag (str or callable) - 是一个字符串(带标签的名称)或接收一个标签名,如果链接应该从标签中提取返回True的函数或False如果他们不应该｡默认为'a' ｡请求(一旦它的下载)作为其第一个参数｡欲了解更多信息,请参阅Passing additional data to callback functions｡
2. attr (str or callable)  - 无论是字符串(带有tag属性的名称) ,或接收到一个属性名称,如果链接应该从中提取返回True的函数或False如果他们不应该｡默认设置为href ｡
3. unique (boolean) - 是一个布尔值,指定是否重复过滤,应用于提取链接｡
3. process_value (callable)  - 
它接收来自扫描标签和属性提取每个值,可以修改该值,并返回一个新的,或返回None完全忽略链接的功能｡如果没有给出, process_value默认是lambda x: x.｡

例如,从这段代码中提取链接:

	<a href="javascript:goToPage('../other/page.html'); return false">Link text</a>
你可以使用下面的这个 process_value函数:

	def process_value(value):
    	m = re.search("javascript:goToPage\('(.*?)'", value)
    	if m:
        	return m.group(1)
