Flask 扩展
================

Flask 扩展多方面地扩充了 Flask 功能。例如它们添加了数据库支持以及其它常见任务。

查找扩展
------------------

Flask 扩展被列出在 `Flask Extension Registry`_  上并且能够用 ``easy_install`` 或者 ``pip`` 下载。
如果你把一个 Flask 扩展作为依赖添加到你的 ``requirements.rst`` 或者 ``setup.py`` 文件，它们通常用一个简单的
命令安装或者在应用安装时被安装。

使用扩展
----------------

扩展通常是有展示如何使用它的文档。扩展如何支持行为上不存在一般的规则，但是它们是从常见的位置导入的。
如果你有一个称为 ``Flask-Foo`` 或者 ``Foo-Flask`` 的扩展，这将是永远导入从 ``flask.ext.foo`` 中::

    from flask.ext import foo

Flask 0.8 以前的 Flask
----------------------

如果你正在使用 Flask 0.7 或者更早的版本，:data:`flask.ext` 是不存在的，相反你必须从 ``flaskext.foo`` 或者 ``flask_foo``
上导入，这取决于扩展的如何分发的。如果你想要开发一个支持 Flask 0.7 或者更早的版本的应用，你应该仍然从 :data:`flask.ext` 包中导入
扩展。我们提供一个兼容模版来在 Flask 的老版本中提供这个包。你可以从 github: `flaskext_compat.py`_ 上下载它。

这里是使用它的方式::

    import flaskext_compat
    flaskext_compat.activate()

    from flask.ext import foo

一旦激活了 ``flaskext_compat`` 模块，就会存在 :data:`flask.ext` ，并且你可以从那里开始导入。

.. _Flask Extension Registry: http://flask.pocoo.org/extensions/

.. _flaskext_compat.py: https://github.com/mitsuhiko/flask/raw/master/scripts/flaskext_compat.py
