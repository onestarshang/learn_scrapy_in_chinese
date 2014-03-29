#Logging#

Scrapy提供了一种日志处理机制，这可以通过[scrapy.log](http://doc.scrapy.org/en/latest/topics/logging.html#module-scrapy.log)模块来使用。当前底层是用[Twisted loggin](http://twistedmatrix.com/projects/core/documentation/howto/logging.html)来实现的，但是这个可能在未来会改变。

这种日志服务必须明确的通过[scrapy.log.start()](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.start)方法来启用。

###Log levels###

Scrapy提供了5种日志级别：

- [CRITICAL](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.CRITICAL) - 对于严重错误
- [ERROR](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.ERROR) - 对于一般错误
- [WARNING](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.WARNING) - 对于警告信息
- [INFO](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.INFO) - 对于提示信息
- [DEBUG](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.DEBUG) - 对于调试信息
 
###How to set the log level###

你可以通过-loglevel/-l命令行选项或者[LOG_LEVEL](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_LEVEL )配置来设置日志级别。

###How to log messages###

这里是一个使用WARNING日志级别来展示如何记录日志的例子。

    from scrapy import log
    log.msg("This is a warning", level=log.WARNING)

###Logging from Spiders###

在编写爬虫记录日志时推荐的方式是通过Spider [log()](http://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spider.Spider.log)方法，此方法已经对[scrapy.log.msg()](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.msg)方法的spider参数进行了赋值，其他参数直接传递给了[msg()](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.msg)函数。

###scrapy.log.module###

scrapy.log.start(logfile=None, loglevel=None, logstdout=None)

启动日志处理机制。此方法必须在实际记录任何日志信息之前被调用，否则，在调用之前已经记录的信息会被遗失掉。

Parameters:
- logfile(str) - 用来日志输出到文件路径。如果忽略此参数，则会使用[LOG_FILE](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_FILE)设置。如果两个都没有设置，那么日志将会被输出到标准错误输出。
- loglevel - 最低的日志级别。可用的值有[CRITICAL](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.CRITICAL)，[ERROR](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.ERROR)，[WARNING](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.WARNING)，[INFO](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.INFO)和[DEBUG](http://doc.scrapy.org/en/latest/topics/logging.html#scrapy.log.DEBUG)。
- logstdout(boolean) - 如果为True，所有你应用程序到标准输出（和错误信息）都会被记录为日志。比如说，你"print 'hello'"，它将会被记录为日志。如果忽略此参数，将会使用[LOG_STDOUT](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_STDOUT)设置。

scrapy.log.msg(message, level=INFO, spider=None)

记录日志信息。

Parameters:
- message(str) - 要记录到日志信息。
- level - 记录此日志信息到日志级别。看[LOG LEVEL](http://doc.scrapy.org/en/latest/topics/logging.html#topics-logging-levels)。
- spider([Spider](http://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spider.Spider)对象) - 记录此日志信息到spider。这个参数应该总是在记录和spider相关到信息时被使用。


scrapy.log.CRITICAL

  对于critical errors到日志级别。

scrapy.log.ERROR

  对于一般错误到日志级别。

scrapy.log.WARNING

  对于警告信息到日志级别。

scrapy.log.INFO

  对于提示信息到日志级别（推荐在生产部署环境中使用）。

scrapy.log.DEBUG

  对于调试信息到日志级别（推荐在开发环境中使用）。

###Logging settings###

这些设置选项可以在配置日志时使用：

- [LOG_ENABLED](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_ENABLED)
- [LOG_ENCODING](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_ENCODING)
- [LOG_FILE](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_FILE)
- [LOG_LEVEL](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_LEVEL)
- [LOG_STDOUT](http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-LOG_STDOUT)

