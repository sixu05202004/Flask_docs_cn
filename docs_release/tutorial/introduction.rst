.. _tutorial-introduction:

介绍 Flaskr
==================

这里我们将称我们的博客应用为 flaskr，也可以取一个不那么 web 2.0 的名字。基本上我们想要它做如下的事情：

1. 根据配置文件中的认证允许用户登录以及注销。仅仅支持一个用户。
2. 当用户登录后，他们可以添加新的条目，这些条目是由纯文本的标题和 HTML 的正文构成。因为我们信任
   用户这里的 HTML 是安全的。
3. 页面倒序显示所有条目（新的条目在前），并且用户登入后可以在此添加新条目。

我们将在这个应用中直接使用 SQLite 3 因为它足够应付这种规模的应用。对更大的应用使用 `SQLAlchemy`_ 
是十分有意义的，它以一种更智能方式处理数据库连接，允许你一次连接多个不用的关系数据库。
你也可以考虑流行的 NoSQL 数据库，如果你的数据更适合它们。

这是最终应用的一个截图:

.. image:: ../_static/flaskr.png
   :align: center
   :class: screenshot
   :alt: screenshot of the final application

继续浏览 :ref:`tutorial-folders` 。

.. _SQLAlchemy: http://www.sqlalchemy.org/
