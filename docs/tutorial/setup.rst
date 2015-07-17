.. _tutorial-setup:

Step 2: 应用设置代码
==============================

现在我们已经有了数据库模式了，我们可以创建应用的模块了。让我们称为 `flaskr.py` ，并
放置于 `flaskr` 文件夹中。对于初学者来说，我们会添加所有需要的导入像配置的章节中一样。对于小应用，直接把配置放在主模块里，正如我们现在要做的一样，是可行的。然而一个更干净的解决方案就是单独创建 `.ini` 或者 `.py` 文件接着加载或者导入里面的值。

在 `flaskr.py` 中::

    # all the imports
    import sqlite3
    from flask import Flask, request, session, g, redirect, url_for, \
         abort, render_template, flash

    # configuration
    DATABASE = '/tmp/flaskr.db'
    DEBUG = True
    SECRET_KEY = 'development key'
    USERNAME = 'admin'
    PASSWORD = 'default'

下一步我们能够创建真正的应用，接着用同一文件(在 `flaskr.py` 中)中的配置初始化::

    # create our little application :)
    app = Flask(__name__)
    app.config.from_object(__name__)

:meth:`~flask.Config.from_object` 将会寻找给定的对象(如果它是一个字符串，则会导入它)，
搜寻里面定义的全部大写的变量。在我们的这种情况中，配置文件就是我们上面写的几行代码。
你也可以将他们分别存储到多个文件。

通常，从配置文件中加载配置是一个好的主意。这是 :meth:`~flask.Config.from_envvar` 所做的，
用它替换上面的 :meth:`~flask.Config.from_object` ::

    app.config.from_envvar('FLASKR_SETTINGS', silent=True)

这种方法我们可以设置一个名为 :envvar:`FLASKR_SETTINGS` 环境变量来设定一个配置文件载入后是否覆盖默认值。
静默开关告诉 Flask 不去关心这个环境变量键值是否存在。

`secret_key` 是需要为了保持客户端的会话安全。明智地选择该键，使得它难以猜测，尽可能复杂。
调试标志启用或禁用交互式调试。*决不让调试模式在生产系统中启动*，因为它将允许用户在服务器上执行代码！

我们还添加了一个轻松地连接到指定数据库的方法，这个方法用于在请求时打开一个连接，并且在交互式 Python shell  和脚本中也能使用。这对以后很方便。

::

    def connect_db():
        return sqlite3.connect(app.config['DATABASE'])

最后如果我们想要把那个文件当做独立应用来运行，我们只需在服务器启动文件的末尾添加这一行::

    if __name__ == '__main__':
        app.run()

如此我们便可以顺利开始运行这个应用，使用如下命令::

   python flaskr.py

你将会看到一个信息，信息提示你服务器启动的地址，这个地址你能够访问到的。

当你在浏览器中访问服务器获得一个404页面无法找到的错误时，是因为我们还没有任何视图。我们之后再来关注这些。首先我们应该让数据库工作起来。

.. admonition:: 外部可见的服务器

   想要你的服务器公开可见吗？更多的信息请查阅 :ref:`externally visible server <public-server>` 。

继续浏览 :ref:`tutorial-dbinit` 。
