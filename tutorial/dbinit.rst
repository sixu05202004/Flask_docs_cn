.. _tutorial-dbinit:

Step 3: 创建数据库
=============================

如前面所述，Flaskr是一个数据库驱动的应用程序，准确地来说，Flaskr是一个使用关系数据库系统的应用程序。
这样的系统需要一个模式告诉它们如何存储信息。因此在首次启动服务器之前，创建数据库模式是很重要的。

可以通过管道把schema.sql作为sqlite3命令的输入来创建这个模式，命令如下::

    sqlite3 /tmp/flaskr.db < schema.sql

这种方法的缺点是需要安装sqlite3命令，而并不是每个系统都有安装。而且你必 须提供数据库的路径，否则将报错。添加一个函数来对初始化数据库是个不错的想法。

如果你想要这么做，首先你必须从contextlib包中导入 :func:`contextlib.closing` 函数。如果你要使用
Python2.5有必要先启用 `with` 声明( `__future__` 导入必须先于其它的导入)::

    from __future__ import with_statement
    from contextlib import closing

接着我们可以创建一个称为 `init_db` 函数，该函数用来初始化数据库。为此我们可以使用之前定义的 `connect_db` 函数。
只要在 `connect_db` 函数下添加这样的函数::

    def init_db():
        with closing(connect_db()) as db:
            with app.open_resource('schema.sql') as f:
                db.cursor().executescript(f.read())
            db.commit()

:func:`~contextlib.closing` 助手函数允许我们在 `with` 块中保持数据库连接。应用对象的 :func:`~flask.Flask.open_resource` 
方法支持函数式“即开即用”，因此它能够直接在 `with` 块中使用。这个函数从资源位置（你的 `flaskr` 文件夹）中打开一个文件，并且允许你读取它。
我们在这里用它在数据库连接上执行一个脚本。

当我们连接到一个数据库的时候，我们得到一个连接对象（这里称为 `db` ），该连接对象能够提供我们一个游标。在这个游标上有一个执行完整脚本的方法。
最后我们仅仅必须提交更改。SQLite3和其他事务性数据库不会提交，除非你明确告诉它。

现在可以在Python shell里创建数据库，导入并调用刚才的函数::

>>> from flaskr import init_db
>>> init_db()

.. admonition:: 疑难排解

   如果你得到一个表不能被找到的异常后，检查你确实调用了 `init_db` 函数以及你的表名是正确的(比如单数复数)。

请继续浏览 :ref:`tutorial-dbcon` 。
