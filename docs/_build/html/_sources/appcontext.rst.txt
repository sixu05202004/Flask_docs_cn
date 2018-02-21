.. _app-context:

应用上下文
=======================

.. versionadded:: 0.9

Flask 背后的一个设计理念是在代码执行的时候存在两种不同的"状态"。应用隐式地处于模块层时配置状态。
这始于 :class:`Flask` 对象实例化，并且当第一个请求传入时，会隐式地结束。当应用处于这种状态时，有如下假设:

-   程序员能够安全地修改应用对象。
-   之前没有发生过任何请求处理。
-   你需要一个应用对象的引用来修改它，没有魔术代理可以给你一个你正在创建或修改的应用对象的引用。

相比之下，在请求处理的时候。一些其它的规则存在：

-   当请求是活跃的时候，上下文本地对象( :data:`flask.request` 和其它的)指向当前请求。
-   任何代码在任何时候都能够找到这些对象以供使用。

这里有一个第三种情况，有一点点差异。有时，你正在用类似请求处理时方式来与应用交互，
即使并没有活动的请求。想象一下你用交互式 Python shell 与应用交互的情况，或是一个命令行应用的情况。

:data:`~flask.current_app` 上下文本地变量就是应用上下文驱动的。

应用上下文目的
----------------------------------

应用上下文存在的主要原因是，在过去，没有更好的方式来在请求上下文中附加一堆函数，
因为 Flask 设计的支柱之一是你可以在一个 Python 进程中拥有多个应用。

那么代码如何找到“正确的”应用？在过去，我们推荐显式地到处传递应用，但是这导致没有用这种想法设计的库的问题，因为让库实现这种想法太不方便。

解决上述问题的常用方法是使用后面将会提到的 :data:`~flask.current_app` 代理，它被限制在当前请求的应用引用。
既然无论如何在没有请求时创建一个这样的请求上下文是一个没有必要的昂贵操作，那么就引入了应用上下文。

创建一个应用上下文
-------------------------------

有两种方式创建一个应用上下文。第一种是隐式的方式：无论何时一个请求上下文被压栈， 一个应用上下文会并排创建，如果这是必须的。由于这样的结果，你可以忽略应用上下文的存在，除非你需要它。

第二种方式是显式地使用 :meth:`~flask.Flask.app_context` 方法::

    from flask import Flask, current_app

    app = Flask(__name__)
    with app.app_context():
        # within this block, current_app points to app.
        print current_app.name

如果 ``SERVER_NAME`` 被配置话，应用上下文也能用于 :func:`~flask.url_for` 函数。这也允许你即使在没有请求的情况下生成 
URLs。

应用上下文的局部变量
-----------------------

应用上下文会按需创建并销毁。它不会在线程间移动，并且也不会在请求间共享。如此，它是一个存储数据库连接信息或是别的东西的最佳位置。内部的栈对象称为 :data:`flask._app_ctx_stack`。
扩展可以在栈的最顶端自由储存信息，前提是使用唯一的名称，相反 :data:`flask.g` 对象是为用户代码保留。

关于此更多的信息，请看 :ref:`extension-dev`。

应用上下文的用法
-----------------

应用上下文通常是用来缓存那些用来请求之前创建的或者请求使用情况下的资源。例如数据库连接是注定要使用应用上下文。
存储的东西时应该为应用程序上下文选择唯一的名称，因为这是一个 Flask 应用和扩展之间共享的地方。

最常见的用法是把资源管理划分成两部分：

1.  一个隐式的缓存上下文的资源。
2.  一个基于资源释放的上下文销毁。

一般来说，``get_X()`` 函数用于创建资源 ``X``，如果 ``X`` 不存在的情况下否则会返回同样的资源，
``teardown_X()`` 函数注册作为销毁处理器。

这是个连接数据库的例子::

    import sqlite3
    from flask import g

    def get_db():
        db = getattr(g, '_database', None)
        if db is None:
            db = g._database = connect_to_database()
        return db

    @app.teardown_appcontext
    def teardown_db(exception):
        db = getattr(g, '_database', None)
        if db is not None:
            db.close()

第一次调用 ``get_db()`` 时，连接将会被建立。建立的过程中隐式地使用了一个 :class:`~werkzeug.local.LocalProxy` 类::

    from werkzeug.local import LocalProxy
    db = LocalProxy(get_db)

这样，用户就可以通过 ``get_db()`` 来直接访问 ``db`` 了。
