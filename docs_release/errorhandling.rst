.. _application-errors:

记录应用程序错误
==========================

.. versionadded:: 0.3

应用程序会失败，服务器会失败。迟早你会看到在生产模式中的一个异常。即使你的代码是 100% 正确，
你仍然将会不时地看到异常。为什么？因为涉及的所有一切都会出现异常。这里是一些完美代码会导致服务器错误的情况：

-   客户端提前中断请求，应用仍在读取请求传入的数据。
-   数据库服务器超载，无法处理查询。
-   一个文件系统已满。
-   一个硬件崩溃。
-   后端服务器超载。
-   您正在使用的一个库中的程序错误。
-   到另一个系统的服务器的网络连接失败。

而这仅仅是你可能会面临的问题一个小样本。因此我们该如何处理这类的问题？默认情况下，
如果你的应用程序在生产模式下运行的话，Flask 会显示一个非常简单的页面并记录异常到 :attr:`~flask.Flask.logger`。

但是你有更多的可做的，我们将介绍一些更好的用来处理错误的设置。

错误邮件
-----------

如果你的应用在生产模式下运行（会在你的服务器上做），默认情况下，你不会看见任何日志消息。为什么会这样？
Flask 试图成为一个零-配置框架。如果没有配置日志会被放在哪里了？猜测不是一个好主意因为可能性太多，
它猜测的位置可能不是一个用户有权限创建日志文件的地方。同样，对于大部分小型应用没有人会去看日志。

事实上，我现在向你保证，如果你给应用程序错误配置一个日志文件，你将永远不会看见它，除非在调试问题时用户向你报告。
你需要的应是异常发生时的邮件，然后你会得到一个警报，并做些什么。

Flask 使用 Python 内置的日志系统，它实际上可以发送错误邮件，这可能是你想要什么。这里是如何配置 Flask 日志向你发送异常邮件::

    ADMINS = ['yourname@example.com']
    if not app.debug:
        import logging
        from logging.handlers import SMTPHandler
        mail_handler = SMTPHandler('127.0.0.1',
                                   'server-error@example.com',
                                   ADMINS, 'YourApplication Failed')
        mail_handler.setLevel(logging.ERROR)
        app.logger.addHandler(mail_handler)

那么刚刚发生了什么了？我们创建一个新的 :class:`~logging.handlers.SMTPHandler` ，它将在监听 ``127.0.0.1`` 
的邮件服务器上以发件人 *server-error@example.com* 以及主题 "YourApplication Failed" 向所有的 `ADMINS`  发送邮件。
如果你的邮件服务器需要认证的话，这些功能可能提供。对此功能请查阅文档 :class:`~logging.handlers.SMTPHandler` 。

我们同样告诉处理程序只发送错误以及更重要的信息。因为我们当然不希望收到关于一些警告或者其它一些可能发生在请求处理的日志的邮件。

在生产模式上运行之前，请查阅 :ref:`logformat` 把更多的信息加入到错误邮件中。这会让你少走弯路。


记录到一个文件中
-----------------

即使你能收到邮件，也可能希望记录警告。当调试问题的时候，收集更多的信息是个好主意。
请注意 Flask 自身核心系统是不会发出任何警告的，因此在不寻常的事情发生时发出警告就是你的责任。

在日志系统的框架外提供了一些处理程序，但它们对记录基本错误并不是都有用。最让人感兴趣的可能是下面的几个：

-   :class:`~logging.FileHandler` - 在文件系统上记录信息。
-   :class:`~logging.handlers.RotatingFileHandler` - 在文件系统上记录信息到一个文件上，
                                                     达到一定数量的消息后会切割。
-   :class:`~logging.handlers.NTEventLogHandler` - 记录到 Windows 系统中的系 统事件日志。
                                                   如果你在 Windows 上做开发，这就是你想要用的。
-   :class:`~logging.handlers.SysLogHandler` - 发送日志到 UNIX 系统日志。

当你选择了日志处理程序，像前面对 SMTP 处理程序做的那样，只要确保使用一个低级的设置（我推荐 `WARNING` ）::

    if not app.debug:
        import logging
        from themodule import TheHandlerYouWant
        file_handler = TheHandlerYouWant(...)
        file_handler.setLevel(logging.WARNING)
        app.logger.addHandler(file_handler)

.. _logformat:

控制日志格式
--------------------------

默认情况下的处理程序将只写消息字符串到一个文件，或向您发送该消息作为邮件。一个日志记 录应存储更多的信息，这使得配置你的日志记录器包含那些信息很重要，如此你会对错误发生的原因，
还有更重要的：错误在哪发生，有更好的了解。

格式可以从一个格式化字符串实例化。需要注意的是的 traceback 信息被自动地加入到日志条目，
你不需要在日志格式的格式化字符串中去做。

这些一些配置的例子：

邮件
`````

::

    from logging import Formatter
    mail_handler.setFormatter(Formatter('''
    Message type:       %(levelname)s
    Location:           %(pathname)s:%(lineno)d
    Module:             %(module)s
    Function:           %(funcName)s
    Time:               %(asctime)s

    Message:

    %(message)s
    '''))

文件记录
````````````

::

    from logging import Formatter
    file_handler.setFormatter(Formatter(
        '%(asctime)s %(levelname)s: %(message)s '
        '[in %(pathname)s:%(lineno)d]'
    ))


复杂的日志格式
``````````````````````

这里是一个有用的格式化字符的格式变量列表。注意这个列表并不完整，完整的列表翻阅 :mod:`logging` 包的官方文档。

.. tabularcolumns:: |p{3cm}|p{12cm}|

+------------------+----------------------------------------------------+
| 格式             | 描述                                               |
+==================+====================================================+
| ``%(levelname)s``| 消息文本的日志记录级别                             |
|                  | (``'DEBUG'``, ``'INFO'``, ``'WARNING'``,           |
|                  | ``'ERROR'``, ``'CRITICAL'``)。                     |
+------------------+----------------------------------------------------+
| ``%(pathname)s`` | 发起日志调用（如果可用）的源文件的完整路径名。     |
+------------------+----------------------------------------------------+
| ``%(filename)s`` | 路径中的文件名部分 。                              |
+------------------+----------------------------------------------------+
| ``%(module)s``   | 模块（文件名的名称部分）。                         |
+------------------+----------------------------------------------------+
| ``%(funcName)s`` | 包含日志调用的函数名 。                            |
+------------------+----------------------------------------------------+
| ``%(lineno)d``   | 日志记录调用所在的源文件行的行号（如果可用）。     |
+------------------+----------------------------------------------------+
| ``%(asctime)s``  | LogRecord创建时可读的时间。默认的形式是            |
|                  | ``"2003-07-08 16:49:45,896"``                      |
|                  | (逗号后的数字时间的毫秒部分)。通过继承格式并且重载 |
|                  | :meth:`~logging.Formatter.formatTime` 方法改变。   |
+------------------+----------------------------------------------------+
| ``%(message)s``  | 记录的消息，记为 ``msg % args`` 。                 |
+------------------+----------------------------------------------------+

如果你想深度定制日志格式，你可以继承格式。格式有三个需要关注的方法：

:meth:`~logging.Formatter.format`:
    处理实际格式。
    传入一个 :class:`~logging.LogRecord` 对象且必须返回格式化的字符串。
:meth:`~logging.Formatter.formatTime`:
    为格式化 `asctime` 而调用。
    如果你想要一个不同的时间格式你可以重载这个方法。
:meth:`~logging.Formatter.formatException`
    为格式化异常而调用。传入一个 :attr:`~sys.exc_info` 元组且必须返回一个字符串。
    默认的是足够好，你不必重载它了。  

更多信息，请查阅官方文档。


其它的库
---------------

目前为止我们只配置了应用程序本身建立的日志记录器。同样其他库也会自己记录日志。比如，
SQLAlchemy 在自己核心代码中大量使用了日志。尽管在 :mod:`logging` 包中有一种方式一次性配置所有日志，
我不建议使用它。可能存在一种情况，当你想 要在同一个 Python 解释器中并排运行多个独立的应用时，
则不可能对它们的日志记录器做不同的设置。

作为替代，我推荐你找出你有兴趣的日志记录器，用 :func:`~logging.getLogger` 函数来获取日志记录器，并且遍历它们来附加处理程序::

    from logging import getLogger
    loggers = [app.logger, getLogger('sqlalchemy'),
               getLogger('otherlibrary')]
    for logger in loggers:
        logger.addHandler(mail_handler)
        logger.addHandler(file_handler)


调试应用程序错误
============================

对于生产应用，按照 :ref:`application-errors` 来配置应用程序日志和通知。
这个章节讲述了调试部署配置和深入一个功能强大的 Python 调试器的要点。


有疑问时，手动运行
---------------------------

在配置你的应用到生产时遇到了问题？如果你拥有主机的 shell 权限，验证你是否可以在部署环境中手动用 shell 运行你的应用。
确保在同一用户账户下运行配置好的部署来解决权限问题。你可以设置 debug=True 来使用 Flask 内置的开发服务器，这在 捕获配置问题的时候非常有效，但是 **请确保在可控环境下临时地这么做**。 不要在生产环境中使用 debug=True 运行。


.. _working-with-debuggers:

使用调试器
----------------------

为了能够更加深入，可能会跟踪代码的执行，Flask 提供了一个框架外的调试器(请看 :ref:`debug-mode`)。
如果你想用其它的 Python 调试器，请注意相互的调试器接口。为了使用你喜爱的调试器你必须设置一些选项：

* ``debug``        - 是否开启调试模式并捕获异常
* ``use_debugger`` - 是否使用内部的 Flask 调试器
* ``use_reloader`` - 是否在异常时重新载入并创建子进程

``debug`` 必须为 True (即，异常必须捕获异常)来允许其它的两个选项设置为任何值。

如果你使用 Aptana/Eclipse 调试，你将需要设置 ``use_debugger`` 和 ``use_reloader`` 为False。

一个可能有用的配置模式就是在你的 config.yaml 中设置为如下(当然，自行更改为适用你应用的)::

   FLASK:
       DEBUG: True
       DEBUG_WITH_APTANA: True

接着在应用程序的入口处，你可以写成这样::

   if __name__ == "__main__":
       # To allow aptana to receive errors, set use_debugger=False
       app = create_app(config="config.yaml")

       if app.debug: use_debugger = True
       try:
           # Disable Flask's debugger if external debugger is requested
           use_debugger = not(app.config.get('DEBUG_WITH_APTANA'))
       except:
           pass
       app.run(use_debugger=use_debugger, debug=app.debug,
               use_reloader=use_debugger, host='0.0.0.0')

