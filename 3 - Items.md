#Items#

###Items###

爬站的最主要目的是从非结构化的数据尤其是网页中提取结构化的数据。Scrapy提供的Item类的目的就在于此。

Item对象是一个简单的存储收集来的数据的容器。它提供类字典的API，用简单的语法声明其可用字段。

###声明Items###

通过简单的定义类的语法和Field对象来声明Items，例如：

<pre><code>from scrapy.item import Item, Field
 
class Product(Item):
    name = Field()
    price = Field()
    stock = Field()
    last_updated = Field(serializer=str)</code></pre>
###Item Fields###

Field对象来指定每个字段的元数据。例如，上例中last_updated中传入的参数表示用于序列化的方法。

你可以为每个字段指定不同的元数据（名称）。对于通过filed对象接受的值没有限制，由于这个原因，无法列出所有可用元数据键的参考列表。通过Field对象定义的键可以用于不同组件，并且只有这些组件知道有哪些键。根据你自己的需求，可以定义任何和使用任何通过Field定义的键。Filed对象的主要目的是提供一种可以在同一个地方定义所有元数据字段的途径。通常情况下，那些行为需要依赖于每个字段的组件，是用每种特定的filed键来确定对应的行为的。你必须参考文档来了解那些元数据键可以用于那些组件。

重要的是Field对象不止用于声明类的属性（以item["xxx"]方式访问），也可以通过Item.field的方式访问。

###使用Items###

这有几个Item的常用方法，通过上面定义的Product演示。你会注意到这个API和字典的API很像。

####创建Item####

<pre><code>>>> product = Product(name='Desktop PC', price=1000)
>>> print product
Product(name='Desktop PC', price=1000)</code></pre>
####获取值####

<pre><code>>>> product['name']
Desktop PC
>>> product.get('name')
Desktop PC
 
>>> product['price']
1000
 
>>> product['last_updated']
Traceback (most recent call last):
...
KeyError: 'last_updated'
 
>>> product.get('last_updated', 'not set')
not set
 
>>> product['lala'] # getting unknown field
Traceback (most recent call last):
...
KeyError: 'lala'
 
>>> product.get('lala', 'unknown field')
'unknown field'
 
>>> 'name' in product # is name field populated?
True
 
>>> 'last_updated' in product # is last_updated populated?
False
 
>>> 'last_updated' in product.fields # is last_updated a declared field?
True
 
>>> 'lala' in product.fields # is lala a declared field?
False</code></pre>
####赋值####

<pre><code>>>> product['last_updated'] = 'today'
>>> product['last_updated']
today
 
>>> product['lala'] = 'test' # setting unknown field
Traceback (most recent call last):
...
KeyError: 'Product does not support field: lala'</code></pre>
####访问所有值####

可以像字典一样访问所有的值：

<pre><code>>>> product.keys()
['price', 'name']
 
>>> product.items()
[('price', 1000), ('name', 'Desktop PC')]</code></pre>
####其他常用用法####

复制items：

<pre><code>>>> product2 = Product(product)
>>> print product2
Product(name='Desktop PC', price=1000)
 
>>> product3 = product2.copy()
>>> print product3
Product(name='Desktop PC', price=1000)</code></pre>
items转换为字典：

<pre><code>>>> dict(product) # create a dict from all populated values
{'price': 1000, 'name': 'Desktop PC'}</code></pre>
字典转换为items：

<pre><code>>>> Product({'name': 'Laptop PC', 'price': 1500})
Product(price=1500, name='Laptop PC')
 
>>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
Traceback (most recent call last):
...
KeyError: 'Product does not support field: lala'</code></pre>
###扩展Itmes###

通过继承你最初定义的的Item类来扩展Item。

例如：

<pre><code>class DiscountedProduct(Product):
    discount_percent = Field(serializer=str)
    discount_expiration_date = Field()</code></pre>
你也可以通过使用之前定义的元数据字段和增加一些值或者改变一些值来扩展Field字段，例如：

<pre><code>class SpecificProduct(Product):
    name = Field(Product.fields['name'], serializer=my_serializer)</code></pre>
上述例子保持其他的数据不变，只为name字段添加了serializer元数据键。

###Item对象###

*class scrapy.item.Item([arg])*

返回新的初始化的item，参数可选。

items继承了标准字典的API，包括它的构造函数。items提供的唯一附加属性就是：

*fields*

一个包含所有item中所有（不仅仅是已经被赋值的）的字段的字典。键是item中字段名，值是用于item声明时的Fields对象。

###Field对象###

*class scrapy.item.Field([arg])*

Field在这里其实就是python内置字典类的别称，并且不提供任何其他的方法或属性。换句话说就是一个纯粹的python字典，把它分离出来用于item语法声明中定义属性的。
