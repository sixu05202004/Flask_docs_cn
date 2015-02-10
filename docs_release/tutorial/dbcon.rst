.. _tutorial-dbcon:

Step 4: 请求数据库连接
------------------------------------

现在我们知道了怎样建立数据库连接以及在脚本中使用这些连接，但是我们如何能优雅地在请求中这么做？
所有我们的函数中需要数据库连接，因此在请求之前初始化它们，在请求结束后自动关闭他们就很有意义。

Flask 允许我们使用 :meth:`~flask.Flask.before_request`，:meth:`~flask.Flask.after_request` 和 
:meth:`~flask.Flask.teardown_request` 装饰器来实现这个功能::

    @app.before_request
    def before_request():
        g.db = connect_db()

    @app.teardown_request
    def teardown_request(exception):
        g.db.close()

使用 :meth:`~flask.Flask.before_request` 装饰器的函数会在请求之前被调用而且不带参数。使用
:meth:`~flask.Flask.after_request` 装饰器的函数会在请求之后被调用且传入将要发给客户端的响应。
它们必须返回那个响应对象或是不同的响应对象。但当异常抛出时，它们不一定会被执行，
这时可以使用 :meth:`~flask.Flask.teardown_request` 装饰器。它装饰的函数将在响应构造后执行，
并不允许修改请求，返回的值会被忽略。如果在请求已经被处理的时候抛出异常，它会被传递到每个函数，
否则会传入一个 `None` 。

我们把当前的数据库连接保存在 Flask 提供的 :data:`~flask.g` 特殊对象中。这个对象只能保存一次请求的信息，
并且在每个函数里都可用。不要用其它对象来保存信息，因为在多线程环境下将不可行。特殊的对象 :data:`~flask.g` 在后台有一些神 奇的机制来保证它在做正确的事情。

请继续浏览 :ref:`tutorial-views` 。

.. hint:: 我该把代码放哪里？

   如果你一直遵循本教程，你可能会问从这步到下一步，代码放在什么地方。逻辑上应该按照模块来组织函数，
   把你新的 ``before_request`` 和 ``teardown_request`` 装饰的函数放在之前的 ``init_db`` 函数下面（逐行遵照教程）。

   如果你需要短时间找到你的思路，请看看 `example source`_ 是如何组织的。在 Flask 中，你可以把所有的应用代码放到单个 Python 
   模块中。但你无需这么做，并且在你的应用 :ref:`grows larger <larger-applications>` 的时候，这显然不是个好主意。   

.. _example source:
   http://github.com/mitsuhiko/flask/tree/master/examples/flaskr/
