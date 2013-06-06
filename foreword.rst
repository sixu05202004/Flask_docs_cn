前言
========

请在开始使用Flask之前阅读本文。也希望本文能够回答关于Flask项目的初衷以及目标，以及flask适用的场景（情境）等问题。

什么是“微”？
------------

“微”(“Micro”)并不是意味着把整个Web应用放入到一个Python文件，尽管确实可以这么做。当然“微”(“Micro”)也不是意味Flask的功能上是不足的。微框架中的“微”(“Micro”)是指Flask旨在保持代码简洁且易于扩展。Flask不会为你做太多的选择，例如选择什么样的数据库。Flask为你做的是很容易修改的，比如选择什么样的模版引擎。其它的一切取决于你，因此Flask能满足你所需要的。

默认情况下，Flask并不包含数据库抽象层，表单验证或者任何其它现有的库(Django)能够处理的。相反，Flask支持扩展，这些扩展能够添加功能到你的应用，像是Flask本身实现的一样。众多的扩展提供了数据库集成，表单验证，上传处理，多种开放的认证技术等功能。Flask可能是“微”型的，但是已经能够在各种各样的需求中生产使用。

配置和约定
-----------------------------

Flask有许多带有合理默认值的配置项，也遵循一些惯例。例如：按惯例，模板和静态文件存储在应用 Python 源代码树下的子目录中，而这是可以改变的，你通常不必这么做，尤其是在刚开始的时候。

与Flask共同成长
------------------

一旦你的Flask项目搭建以及运行起来，你会发现在社区中有大量可用的扩展集成到你的生产环境项目中来。Flask 核心团队会审阅这些扩展，确保经过验证过的扩展在未来版本中仍能使用。

随着你的代码库的增长，你能够自由地做出决定是恰当的设计为您的项目

As your codebase grows, you are free to make the design decisions appropriate
for your project.  Flask will continue to provide a very simple glue layer to
the best that Python has to offer.  You can implement advanced patterns in
SQLAlchemy or another database tool, introduce non-relational data persistence
as appropriate, and take advantage of framework-agnostic tools built for WSGI,
the Python web interface.

Flask includes many hooks to customize its behavior. Should you need more
customization, the Flask class is built for subclassing. If you are interested
in that, check out the :ref:`becomingbig` chapter.  If you are curious about
the Flask design principles, head over to the section about :ref:`design`.

Continue to :ref:`installation`, the :ref:`quickstart`, or the
:ref:`advanced_foreword`.
