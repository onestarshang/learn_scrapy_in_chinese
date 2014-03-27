#Logging#

Scrapy提供了一种日志处理机制，这可以通过scrapy.log模块来使用。当前底层是用Twisted loggin来实现的，但是这个可能在未来会改变。

这种日志服务必须明确的通过scrapy.log.start()方法来启用。

###Log levels###

Scrapy提供了5种日志级别：

- CRITICAL - 对于严重错误
- ERROR - 对于一般错误
- WARNING - 对于警告信息
- INFO - 对于提示信息
- DEBUG - 对于调试信息
 
###How to set the log level###

你可以通过-loglevel/-l命令行选项或者LOG_LEVEL配置来设置日志级别。

###How to log messages###

这里是一个使用WARNING日志级别来展示如何记录日志的例子。

    from scrapy import log
    log.msg("This is a warning", level=log.WARNING)
