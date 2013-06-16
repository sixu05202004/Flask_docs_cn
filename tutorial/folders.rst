.. _tutorial-folders:

Step 0: 创建文件夹
============================

在我们开始之前，让我们为这个应用创建需要的文件夹::

    /flaskr
        /static
        /templates

The `flaskr` folder is not a python package, but just something where we
drop our files.  Directly into this folder we will then put our database
schema as well as main module in the following steps.  The files inside
the `static` folder are available to users of the application via `HTTP`.
This is the place where css and javascript files go.  Inside the
`templates` folder Flask will look for `Jinja2`_ templates.  The
templates you create later in the tutorial will go in this directory.

继续浏览 :ref:`tutorial-schema` 。

.. _Jinja2: http://jinja.pocoo.org/2/
