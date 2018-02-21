.. _tutorial-dbinit:

Step 3: 创建数据库
=============================

如前面所述，Flaskr 是一个数据库驱动的应用程序，准确地来说，Flaskr 是一个使用关系数据库系统的应用程序。
这样的系统需要一个模式告诉它们如何存储信息。因此在首次启动服务器之前，创建数据库模式是很重要的。

可以通过管道把 schema.sql 作为 sqlite 3 命令的输入来创建这个模式，命令如下::

    sqlite3 /tmp/flaskr.db < schema.sql

这种方法的缺点是需要安装 sqlite 3 命令，而并不是每个系统都有安装。而且你必须提供数据库的路径，否则将报错。添加一个函数来对初始化数据库是个不错的想法。

如果你想要这么做，首先你必须从 contextlib 包中导入 :func:`contextlib.closing` 函数。并且在 `flaskr.py` 文件中添加如下的内容::

    from contextlib import closing

接着我们可以创建一个称为 `init_db` 函数，该函数用来初始化数据库。为此我们可以使用之前定义的 `connect_db` 函数。
只要在 `connect_db` 函数下添加这样的函数::

    def init_db():
        with closing(connect_db()) as db:
            with app.open_resource('schema.sql') as f:
                db.cursor().executescript(f.read())
            db.commit()

:func:`~contextlib.closing` 助手函数允许我们在 `with` 块中保持数据库连接可用。
应用对象的 :func:`~flask.Flask.open_resource` 方法在其方框外也支持这个功能，
因此可以在 `with` 块中直接使用。这个函数从资源位置（你的 `flaskr` 文 件夹）中打开一个文件，并且允许你读取它。我们在这里用它在数据库连接上执行一个脚本。

当我们连接到数据库时会得到一个数据库连接对象（这里命名它为 `db` ），这个对象提供给我们一个数据库指针。指针上有一个可以执行完整脚本的方法。最后我们不显式地提交更改， 
SQLite 3 或者其它事务数据库不会这么做。

现在可以在 Python shell 里创建数据库，导入并调用刚才的函数::

>>> from flaskr import init_db
>>> init_db()

.. admonition:: Troubleshooting

   如果你后面得到一个表不能找到的异常，请检查你是否调用了 `init_db` 函数以及你的表名是否正确 (例如： singular vs. plural)。

请继续浏览 :ref:`tutorial-dbcon` 。
