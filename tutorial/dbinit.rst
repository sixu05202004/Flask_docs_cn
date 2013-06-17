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

The :func:`~contextlib.closing` helper function allows us to keep a
connection open for the duration of the `with` block.  The
:func:`~flask.Flask.open_resource` method of the application object
supports that functionality out of the box, so it can be used in the
`with` block directly.  This function opens a file from the resource
location (your `flaskr` folder) and allows you to read from it.  We are
using this here to execute a script on the database connection.

When we connect to a database we get a connection object (here called
`db`) that can give us a cursor.  On that cursor there is a method to
execute a complete script.  Finally we only have to commit the changes.
SQLite 3 and other transactional databases will not commit unless you
explicitly tell it to.

现在可以在Python shell里创建数据库，导入并调用刚才的函数::

>>> from flaskr import init_db
>>> init_db()

.. admonition:: Troubleshooting

   If you get an exception later that a table cannot be found check that
   you did call the `init_db` function and that your table names are
   correct (singular vs. plural for example).

请继续浏览 :ref:`tutorial-dbcon` 。
