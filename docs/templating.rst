模版
=========

Flask 利用 Jinja2 的模板引擎。当然你是自由地使用不同的模版引擎，但是你仍然必须安装 Jinja2 为了运行
Flask 本身。这个要求是必要的，为了丰富的扩展可用。所有的扩展是依赖于 Jinja2 的存在。

本节仅仅给出一个 Jinja2 如何融入 Flask 的快速介绍。如果想要关于模版引擎语法自身的信息，查看官方的 
`Jinja2 Template Documentation <http://jinja.pocoo.org/2/documentation/templates>`_  获取更多的信息。

Jinja 配置
-----------

除非定制，Flask 中 Jinja2 配置如下：

-   所有以 ``.html`` ，``.htm`` ，``.xml`` 以及 ``.xhtml`` 结尾的模版开启自动转义。
-   一个模板可以用 ``{% autoescape %}`` 标签选择开关自动转义。
-   Flask 在 Jinja2 上下文中插入了几个全局函数和助手，另外还有一些目前默认的值。

标准上下文
----------------

下面全局变量默认在 Jinja2 模版中可用:

.. data:: config
   :noindex:

   当前配置对象 (:data:`flask.config`)

   .. versionadded:: 0.6

   .. versionchanged:: 0.10
     现在一直可用，即使是导入的模版。

.. data:: request
   :noindex:

   当前请求对象 (:class:`flask.request`)

.. data:: session
   :noindex:

   当前会话对象 (:class:`flask.session`)

.. data:: g
   :noindex:

   实现全局变量的请求范围的对象 (:data:`flask.g`)

.. function:: url_for
   :noindex:

   :func:`flask.url_for` 函数。

.. function:: get_flashed_messages
   :noindex:

   :func:`flask.get_flashed_messages` 函数。

.. admonition:: Jinja 上下文行为

   这些变量被添加到上下文变量，它们不是全局变量。不同在于，它们默认不会 在导入模板的上下文中出现。这样做，一方面是考虑到性能，另一方面是为了让事情显式透明。

   对于你着意味着什么？如果你希望导入一个宏，你有两种可能来访问请求对象：

   1.   你显式地传入请求或请求对象的属性作为宏的参数。
   2.   使用 "with context" 导入宏。

   在上下文中导入的方式如下：

   .. sourcecode:: jinja

      {% from '_helpers.html' import my_macro with context %}

标准过滤器
----------------

这些过滤器在 Jinja2 中是可用的，也是 Jinja2 自带的过滤器：

.. function:: tojson
   :noindex:

   这个函数把给定的对象转换成 JSON 表示。如果你要动态生成 JavaScript 这里是一个非常有用的例子。

   注意在 `script` 标签里面转义是不应该发生的，因此你打算在 `script` 标签里面使用它确保用 ``|safe`` 
   禁用转义：

   .. sourcecode:: html+jinja

       <script type=text/javascript>
           doSomethingWith({{ user.username|tojson|safe }});
       </script>

   ``|tojson`` 过滤器会为你恰当地转义斜线。

控制自动转义
------------------------

自动转义的概念是自动为你转义特殊字符。HTML (或者 XML，以及 XHTML)意义下的特殊字符是
``&``, ``>``, ``<``, ``"`` 以及 ``'`` 。因为这些字符在文档中表示它们特定的含义，
如果你想在文本中使用它们，应该把它们替换成相应的“实体”。不这么做的话不仅仅会让用户很难
在文本中使用这么字符，而且会导致安全问题。(请参看 :ref:`xss` )

然而有时间你需要在模版中禁用自动转义。这种情况可能是你想要在页面中显式地插入 HTML，
比如内容来自一个 markdown 到 HTML 转换器的安全的 HTML 输出。

这有三种方式完成这个工作：

-   在 Python 代码中，在传递到模板之前，用 :class:`~flask.Markup` 对象封装 HTML 字符串。这是一般的推荐方法。
-   在模版中，使用 ``|safe`` 过滤器显式地标记一个字符串为安全的 HTML。
-   暂时地禁用自动转义系统。

你可以使用 ``{% autoescape %}`` 块在模板中禁用转义系统：

.. sourcecode:: html+jinja

    {% autoescape false %}
        <p>autoescaping is disabled here
        <p>{{ will_not_be_escaped }}
    {% endautoescape %}

无论你在什么时候做这个，请小心你在块中使用的变量。

.. _registering-filters:

注册过滤器
-------------------

如果你要在 Jinja2 中注册你自己的过滤器，有两种方式。你可以手动地把它们加入到应用的 :attr:`~flask.Flask.jinja_env`，
或者使用 :meth:`~flask.Flask.template_filter` 装饰器。

下面两个例子作用相同，都是反转一个对象::

    @app.template_filter('reverse')
    def reverse_filter(s):
        return s[::-1]

    def reverse_filter(s):
        return s[::-1]
    app.jinja_env.filters['reverse'] = reverse_filter

在使用装饰器的情况下，如果你想以函数名当作过滤器名，参数是可选的。注册之后，你可以在模板中像使用 Jinja2  内置过滤器一样使用你的过滤器，例如你在上下文中有一个名为 `mylist` 的 Python 列表::

    {% for x in mylist | reverse %}
    {% endfor %}


上下文处理器
------------------

Flask 中的上下文处理器自动向模板的上下文中插入新变量。上下文处理器在模板 渲染之前运行，并且可以在模板上下文中插入新值。上下文处理器是一个返回字典的函数。
这个字典的键值将与应用中的所有模板上下文结合::

    @app.context_processor
    def inject_user():
        return dict(user=g.user)

上述的上下文处理器使得一个名为 `user`，值为 `g.user` 的变量在模版中可用。这个例子不是很有意思，因为 `g` 在任何模板中都是可用的，但是它解释了上下文处理器是如何工作的。

变量不仅限于值；一个上下文处理器也可以使函数在模板中可用（由于 Python 允许传递函数）::

    @app.context_processor
    def utility_processor():
        def format_price(amount, currency=u'€'):
            return u'{0:.2f}{1}.format(amount, currency)
        return dict(format_price=format_price)

上面的上下文处理器使得 `format_price` 函数在所有模板中可用::

    {{ format_price(0.33) }}

你也可以构建 `format_price` 为一个模板处理器(参看 :ref:`registering-filters`)，
但这展示了上下文处理器如何传递一个函数。

