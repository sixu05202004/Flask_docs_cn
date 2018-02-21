基于后台作业的 Celery 
=============================

Celery 是一个异步任务队列/基于分布式消息传递的作业队列，它用 Python 编写。曾经整合进 Flask 但是由于
Celery 3版本的内部重构后整合变得没有必要。本指南用于填补如何在 Flask 中使用 Celery 的空白并且假设你已经
阅读了官方文档 `First Steps with Celery
<http://docs.celeryproject.org/en/master/getting-started/first-steps-with-celery.html>`_ 。

安装 Celery
-----------------

Celery是在 Python Package Index (PyPI) 上，因此它能够用标准的 Python 工具像 ``pip`` 或者 ``easy_install`` 来安装::

    $ pip install celery

配置 Celery
------------------

你首先需要的事情就是一个 Celery 示例，这就是所谓的celery应用。它跟 Flask 中的 :class:`~flask.Flask` 对象
有着异曲同工之妙，不过只为 Celery。因此这个实例作为你想要在 Celery 中所做的一切的入口，像常见任务以及管理 workers，
它必须跟其他模块一样导入。

例如你可以把它放在 ``tasks`` 模块。然而你可以不需要任何配置在 Flask 中使用 Celery ，通过继承任务以及为Flask应用上下文
增加支持，使用Flask配置它会变得智能。

这就是所有在 Flask 中整合 Celery 必要步骤::

    from celery import Celery

    def make_celery(app):
        celery = Celery(app.import_name, broker=app.config['CELERY_BROKER_URL'])
        celery.conf.update(app.config)
        TaskBase = celery.Task
        class ContextTask(TaskBase):
            abstract = True
            def __call__(self, *args, **kwargs):
                with app.app_context():
                    return TaskBase.__call__(self, *args, **kwargs)
        celery.Task = ContextTask
        return celery

函数创建一个新的 Celery 对象，用应用配置中的 broker 来配置 Celery 对象，从Flask中的配置来
配置剩下的 Celery 配置。

小型例子
---------------

使用上面的内容，这就是在Flask中使用 Celery 的小型例子::

    from flask import Flask

    app = Flask(__name__)
    app.config.update(
        CELERY_BROKER_URL='redis://localhost:6379',
        CELERY_RESULT_BACKEND='redis://localhost:6379'
    )
    celery = make_celery(app)


    @celery.task()
    def add_together(a, b):
        return a + b

这个任务现在能够后台运行：

>>> result = add_together.delay(23, 42)
>>> result.wait()
65

运行 Celery Worker
-------------------------

现在如果你已经执行上面的代码，你也许会十分失望的，因为你的 ``.wait()`` 实际上没有任何返回。
这是因为你也必须运行 celery 。你可以通过运行 celery 作为一个 worker 来实现::

    $ celery -A your_application worker

``your_application`` 字符串必须指向创建 `celery` 对象的应用包或者模块。

