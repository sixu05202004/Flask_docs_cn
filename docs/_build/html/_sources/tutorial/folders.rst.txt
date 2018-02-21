.. _tutorial-folders:

Step 0: 创建文件夹
============================

在我们开始之前，让我们为这个应用创建需要的文件夹::

    /flaskr
        /static
        /templates

`flaskr` 文件夹不是一个 Python 的包，只是我们放置文件的地方。
在接下来的步骤中我们会直接把数据库模式和主模块放在这个文件夹中。应用的使用用户可以通过
`HTTP` 访问 `static` 文件夹中的文件。这里也是 css 和 javascript 文件存放位置。Flask 将会在
`templates` 文件夹中寻找 `Jinja2`_ 模版。在本教程后面创建的模板将会在这个文件夹中。

继续浏览 :ref:`tutorial-schema` 。

.. _Jinja2: http://jinja.pocoo.org/2/
