.. _quickstart:

快速入门
==========

迫切希望上手？本文提供了一个很好的 Flask 介绍。假设你已经安装 Flask，
如果还没有安装话，请浏览下 :ref:`installation` 。


一个最小的应用
---------------------

一个最小的应用看起来像这样::

    from flask import Flask
    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        return 'Hello World!'

    if __name__ == '__main__':
        app.run()

把它保存成 `hello.py` (或者类似的文件)，然后用 Python 解释器运行它。确保你的应用不叫做 `flask.py`，
因为这会与 Flask 本身冲突。

::

    $ python hello.py
     * Running on http://127.0.0.1:5000/

现在浏览 `http://127.0.0.1:5000/ <http://127.0.0.1:5000/>`_，你会看到你的 `Hello World` 问候。

那么这段代码做了什么？

1. 首先我们导入了类 :class:`~flask.Flask` 。这个类的实例化将会是我们的 WSGI 应用。第一个参数是应用模块的名称。
   如果你使用的是单一的模块（就如本例），第一个参数应该使用 `__name__`。因为取决于如果它以单独应用启动或作为模块导入，
   名称将会不同 （ ``'__main__'`` 对应于实际导入的名称）。获取更多的信息，请阅读 :class:`~flask.Flask` 的文档。
2. 接着，我们创建一个该类的实例。我们传递给它模块或包的名称。这样 Flask 才会知道去哪里寻找模板、静态文件等等。
3. 我们使用装饰器 :meth:`~flask.Flask.route` 告诉 Flask 哪个 URL 才能触发我们的函数。
4. 定义一个函数，该函数名也是用来给特定函数生成 URLs，并且返回我们想要显示在用户浏览器上的信息。
5. 最后我们用函数 :meth:`~flask.Flask.run` 启动本地服务器来运行我们的应用。``if __name__ == '__main__':``
   确保服务器只会在该脚本被 Python 解释器直接执行的时候才会运行，而不是作为模块导入的时候。

请按 control-C 来停止服务器。

.. _public-server:

.. admonition:: 外部可见服务器 

   当你运行服务器，你会注意到它只能从你自己的计算机上访问，网络中其它任何的地方都不能访问。
   这是因为默认情况下，调试模式，应用中的一个用户可以执行你计算机上的任意 Python 代码。

   如果你关闭 `debug` 或者信任你所在网络上的用户，你可以让你的服务器对外可用，只要简单地改变方法
   :meth:`~flask.Flask.run` 的调用像如下这样::

       app.run(host='0.0.0.0')

   这让你的操作系统去监听所有公开的 IP。


.. _debug-mode:

调试模式
----------

:meth:`~flask.Flask.run` 方法是十分适用于启动一个本地开发服务器，但是你需要在修改代码后手动重启服务器。
这样做并不好，Flask 能做得更好。如果启用了调试支持，在代码修改的时候服务器能够自动加载，
并且如果发生错误，它会提供一个有用的调试器。

有两种方式开启调式模式。一种是在应用对象上设置标志位::

    app.debug = True
    app.run()

或者作为 run 的一个参数传入::

    app.run(debug=True)

两种方法效果是一样的。

.. admonition:: Attention

   尽管交互式调试器不能在分叉( forking )环境上工作(这使得它几乎不可能在生产服务器上使用)，
   它依然允许执行任意代码。这使它成为一个巨大的安全风险，因此它 **绝对不能用于生成环境**。

运行中的调试器的截图:

.. image:: _static/debugger.png
   :align: center
   :class: screenshot
   :alt: screenshot of debugger in action

是不是还有其它的调试器？请查看 :ref:`working-with-debuggers`。


路由
-------

现代 Web 应用程序有优雅的 URLs。这能够帮助人们记住 URLs，这点在面向使用慢网络连接的移动设备的应用上有用。
如果用户不必通过点击首页而直接访问想要的页面，很可能他们会喜欢这个页面而且下次再次访问。

正如上面所说， :meth:`~flask.Flask.route` 装饰器是用于把一个函数绑定到一个 URL 上。这有些基本的例子::

    @app.route('/')
    def index():
        return 'Index Page'

    @app.route('/hello')
    def hello():
        return 'Hello World'

但是不仅如此！你可以动态地构造 URL 的特定部分，也可以在一个函数上附加多个规则。

变量规则
``````````````

为了给 URL 增加变量的部分，你需要把一些特定的字段标记成 ``<variable_name>``。这些特定的字段将作为参数传入到你的函数中。当然也可以指定一个可选的转换器通过规则 ``<converter:variable_name>``。
这里有一些不错的例子::

    @app.route('/user/<username>')
    def show_user_profile(username):
        # show the user profile for that user
        return 'User %s' % username

    @app.route('/post/<int:post_id>')
    def show_post(post_id):
        # show the post with the given id, the id is an integer
        return 'Post %d' % post_id

存在如下转换器:

=========== ===========================================
`int`       接受整数
`float`     同 `int` 一样，但是接受浮点数
`path`      和默认的相似，但也接受斜线
=========== ===========================================

.. admonition:: 唯一 URLs / 重定向行为

   Flask 的 URL 规则是基于 Werkzeug 的 routing 模块。
   该模块背后的想法是基于 Apache 和早期的 HTTP 服务器定下先例确保优雅和唯一的 URL。  

   以这两个规则为例::

        @app.route('/projects/')
        def projects():
            return 'The project page'

        @app.route('/about')
        def about():
            return 'The about page'

   虽然它们看起来确实相似，但它们结尾斜线的使用在 URL *定义* 中不同。 
   第一种情况中，规范的 URL 指向 projects 尾端有一个斜线。
   这种感觉很像在文件系统中的文件夹。访问一个结尾不带斜线的 URL 会被 Flask 重定向到带斜线的规范URL去。

   然而，第二种情况的 URL 结尾不带斜线，类似 UNIX-like 系统下的文件的路径名。
   访问结尾带斜线的 URL 会产生一个 404 “Not Found” 错误。

   当用户访问页面时忘记结尾斜线时，这个行为允许关联的 URL 继续工作，
   并且与 Apache 和其它的服务器的行为一致。另外，URL 会保持唯一，有助于避免搜索引擎索引同一个页面两次。


.. _url-building:

构建 URL
````````````

如果它可以匹配 URL，那么 Flask 能够生成它们吗？当然 Flask 能够做到。你可以使用函数
:func:`~flask.url_for` 来针对一个特定的函数构建一个 URL。它能够接受函数名作为第一参数，以及一些关键字参数，
每一个关键字参数对应于 URL 规则的变量部分。未知变量部分被插入到 URL 中作为查询参数。这里有些例子：

>>> from flask import Flask, url_for
>>> app = Flask(__name__)
>>> @app.route('/')
... def index(): pass
...
>>> @app.route('/login')
... def login(): pass
...
>>> @app.route('/user/<username>')
... def profile(username): pass
...
>>> with app.test_request_context():
...  print url_for('index')
...  print url_for('login')
...  print url_for('login', next='/')
...  print url_for('profile', username='John Doe')
...
/
/login
/login?next=/
/user/John%20Doe

(这里也使用了 :meth:`~flask.Flask.test_request_context` 方法，下面会给出解释。这个方法告诉 Flask 表现得像是在处理一个请求，即使我们正在通过 Python 的 shell 交互。
请看下面的解释。 :ref:`context-locals`)。

为什么你愿意构建 URLs 而不是在模版中硬编码？这里有三个好的理由：

1. 反向构建通常比硬编码更具备描述性。更重要的是，它允许你一次性修改 URL， 
   而不是到处找 URL 修改。
2. 构建 URL 能够显式地处理特殊字符和 Unicode 转义，因此你不必去处理这些。
3. 如果你的应用不在 URL 根目录下(比如，在
   ``/myapplication`` 而不在 ``/``)， :func:`~flask.url_for` 将会适当地替你处理好。


HTTP方法
````````````

HTTP (也就说 web 应用协议)有不同的方法来访问 URLs。默认情况下，路由只会响应 `GET` 请求，
但是能够通过给 :meth:`~flask.Flask.route` 装饰器提供 `methods` 参数来改变。这里是些例子::

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        if request.method == 'POST':
            do_the_login()
        else:
            show_the_login_form()

如果使用 `GET` 方法，`HEAD` 方法将会自动添加进来。你不必处理它们。也能确保 `HEAD` 请求
会按照 `HTTP RFC`_ (文档在 HTTP 协议里面描述) 要求来处理， 因此你完全可以忽略这部分 HTTP 规范。
同样地，自从 Flask 0.6 后，`OPTIONS` 也能自动为你处理。

也许你并不清楚 HTTP 方法是什么？别担心，这里有一个 HTTP 方法的快速入门以及为什么它们重要：

HTTP 方法（通常也称为“谓词”）告诉服务器客户端想要对请求的页面 *做* 什么。下面这些方法是比较常见的：

`GET`
    浏览器通知服务器只 *获取* 页面上的信息并且发送回来。这可能是最常用的方法。 

`HEAD`
    浏览器告诉服务器获取信息，但是只对 *头信息* 感兴趣，不需要整个页面的内容。
    应用应该处理起来像接收到一个 `GET` 请求但是不传递实际内容。在 Flask 中你完全不需要处理它，
    底层的 Werkzeug 库会为你处理的。

`POST`
    浏览器通知服务器它要在 URL 上 *提交* 一些信息，服务器必须保证数据被存储且只存储一次。
    这是 HTML 表单通常发送数据到服务器的方法。

`PUT`
    同 `POST` 类似，但是服务器可能触发了多次存储过程，多次覆盖掉旧值。现在你就会问这有什么用，
    有许多理由需要如此去做。考虑下在传输过程中连接丢失：在这种情况下浏览器 和服务器之间的系统可能安全地第二次接收请求，而不破坏其它东西。对于 `POST` 是不可能实现的，因为
    它只会被触发一次。 

`DELETE`
    移除给定位置的信息。

`OPTIONS`
    给客户端提供一个快速的途径来指出这个 URL 支持哪些 HTTP 方法。从 Flask 0.6 开始，自动实现了它。

现在比较有兴趣的是在 HTML4 和 XHTML1，表单只能以 `GET` 和 `POST` 方法来提交到服务器。在 JavaScript 和以后的 HTML 标准中也能使用其它的方法。同时，HTTP 最近变得十分流行，浏览器不再是唯一使用 HTTP 的客户端。比如，许多版本控制系统使用 HTTP。 

.. _HTTP RFC: http://www.ietf.org/rfc/rfc2068.txt

静态文件
------------

动态的 web 应用同样需要静态文件。CSS 和 JavaScript 文件通常来源于此。理想情况下，
你的 web 服务器已经配置好为它们服务，然而在开发过程中 Flask 能够做到。
只要在你的包中或模块旁边创建一个名为 `static` 的文件夹，在应用中使用 `/static` 即可访问。

给静态文件生成 URL ，使用特殊的 ``'static'`` 端点名::

    url_for('static', filename='style.css')

这个文件应该存储在文件系统上称为 ``static/style.css``。

渲染模板
-------------------

在 Python 中生成 HTML 并不好玩，实际上是相当繁琐的，因为你必须自行做好 HTML 转义以保持应用程序的安全。
由于这个原因，Flask 自动为你配置好 `Jinja2
<http://jinja.pocoo.org/2/>`_ 模版。

你可以使用方法 :func:`~flask.render_template` 来渲染模版。所有你需要做的就是提供模版的名称以及你想要作为关键字参数传入模板的变量。这里有个渲染模版的简单例子::

    from flask import render_template

    @app.route('/hello/')
    @app.route('/hello/<name>')
    def hello(name=None):
        return render_template('hello.html', name=name)

Flask 将会在 `templates` 文件夹中寻找模版。因此如果你的应用是个模块，这个文件夹在模块的旁边，如果它是一个包，那么这个文件夹在你的包里面:

**Case 1**: 一个模块::

    /application.py
    /templates
        /hello.html

**Case 2**: 一个包::

    /application
        /__init__.py
        /templates
            /hello.html

对于模板，你可以使用 Jinja2 模板的全部功能。详细信息查看官方的 `Jinja2 Template Documentation
<http://jinja.pocoo.org/2/documentation/templates>`_ 。

这里是一个模版的例子：

.. sourcecode:: html+jinja

    <!doctype html>
    <title>Hello from Flask</title>
    {% if name %}
      <h1>Hello {{ name }}!</h1>
    {% else %}
      <h1>Hello World!</h1>
    {% endif %}

在模版中你也可以使用 :class:`~flask.request`,
:class:`~flask.session` 和 :class:`~flask.g` [#]_ 对象，也能使用函数 :func:`~flask.get_flashed_messages` 。

模版继承是十分有用的。如果想要知道模版继承如何工作的话，请阅读文档
:ref:`template-inheritance` 。基本的模版继承使得某些特定元素（如标题，导航和页脚）在每一页成为可能。

自动转义是开启的，因此如果 `name` 包含 HTML，它将会自动转义。如果你信任一个变量，并且你知道它是安全的
（例如一个模块把 wiki 标记转换到 HTML ），你可以用 :class:`~jinja2.Markup` 类或 ``|safe`` 过滤器在模板中标记它是安全的。
在 Jinja 2 文档中，你会见到更多例子。

这是一个 :class:`~jinja2.Markup` 类如何工作的基本介绍：

>>> from flask import Markup
>>> Markup('<strong>Hello %s!</strong>') % '<blink>hacker</blink>'
Markup(u'<strong>Hello &lt;blink&gt;hacker&lt;/blink&gt;!</strong>')
>>> Markup.escape('<blink>hacker</blink>')
Markup(u'&lt;blink&gt;hacker&lt;/blink&gt;')
>>> Markup('<em>Marked up</em> &raquo; HTML').striptags()
u'Marked up \xbb HTML'

.. versionchanged:: 0.5

   自动转义不再在所有模版中启用。模板中下列后缀的文件会触发自动转义：``.html``, ``.htm``,
   ``.xml``, ``.xhtml``。从字符串加载的模板会禁用自动转义。 

.. [#] 不知道 :class:`~flask.g` 对象是什么？它是可以按你的需求存储信息的东西，更多的信息查看 :class:`~flask.g` 文档以及 
   :ref:`sqlite3`。


接收请求数据
----------------------

对于 web 应用来说，对客户端发送给服务器的数据做出反应至关重要。在 Flask 中由全局对象 :class:`~flask.request` 
来提供这些信息。如果你有一定的 Python 经验，你会好奇这个对象怎么可能是全局的，并且 Flask 是怎么还能保证线程安全。
答案是上下文作用域:


.. _context-locals:

局部上下文
``````````````

.. admonition:: 内幕消息
   
   如果你想要了解它是如何工作以及如何用它实现测试，请阅读本节，否则请跳过本节。

Flask 中的某些对象是全局对象，但不是通常的类型。这些对象实际上是给定上下文的局部对象的代理。
虽然很拗口，但实际上很容易理解。

想象下线程处理的上下文。一个请求传入，web 服务器决定产生一个新线程(或者其它东西，
底层对象比线程更有能力处理并发系统)。当 Flask 开始它内部请求处理时，它认定当前线程是活动的上下文并绑定当前的应用和 WSGI 环境到那 个上下文（线程）。它以一种智能的方法来实现，以致一个应用可以调用另一个应用而不会中断。

所以这对你意味着什么了？如果你是做一些类似单元测试的事情否则基本你可以完全忽略这种情况。
你会发现依赖于请求对象的代码会突然中断，因为没有请求对象。解决方案就是自己创建一个请求并把它跟上下文绑定。
针对单元测试最早的解决方案是使用 :meth:`~flask.Flask.test_request_context` 上下文管理器。结合 `with` 声明，它将绑定一个测试请求来进行交互。这里是一个例子::

    from flask import request

    with app.test_request_context('/hello', method='POST'):
        # now you can do something with the request until the
        # end of the with block, such as basic assertions:
        assert request.path == '/hello'
        assert request.method == 'POST'

另一个可能性就是传入整个 WSGI 环境到 :meth:`~flask.Flask.request_context` 方法::

    from flask import request

    with app.request_context(environ):
        assert request.method == 'POST'

请求对象
``````````````````

请求对象在 API 章节中描述，这里我们不再详细涉及(请看 :class:`~flask.request`)。这里对一些最常见的操作进行概述。
首先你需要从 `flask` 模块中导入它::

    from flask import request

当前请求的方法可以用 :attr:`~flask.request.method` 属性来访问。你可以用 :attr:`~flask.request.form` 属性来访问表单数据
(数据在 `POST` 或者 `PUT` 中传输)。这里是上面提及到两种属性的完整的例子::

    @app.route('/login', methods=['POST', 'GET'])
    def login():
        error = None
        if request.method == 'POST':
            if valid_login(request.form['username'],
                           request.form['password']):
                return log_the_user_in(request.form['username'])
            else:
                error = 'Invalid username/password'
        # the code below this is executed if the request method
        # was GET or the credentials were invalid
        return render_template('login.html', error=error)

如果在 `form` 属性中不存在上述键值会发生些什么？在这种情况下会触发一个特别的 :exc:`KeyError`。
你可以像捕获标准的 :exc:`KeyError` 来捕获它，如果你不这样去做，会显示一个 HTTP 400 Bad Request 错误页面。
所以很多情况下你不需要处理这个问题。

你可以用 :attr:`~flask.request.args` 属性来接收在 URL ( ``?key=value`` ) 中提交的参数::

    searchword = request.args.get('key', '')

我们推荐使用 `get` 来访问 URL 参数或捕获 `KeyError` ，因为用户可能会修改 URL， 
向他们显示一个 400 bad request 页面不是用户友好的。

想获取请求对象的完整的方法和属性清单，请参阅 :class:`~flask.request` 的文档。


文件上传
````````````

你能够很容易地用 Flask 处理文件上传。只要确保在你的 HTML 表单中不要忘记设置属性 ``enctype="multipart/form-data"``，
否则浏览器将不传送文件。

上传的文件是存储在内存或者文件系统上一个临时位置。你可以通过请求对象中 :attr:`~flask.request.files` 属性访问这些文件。
每个上传的文件都会存储在这个属性字典里。它表现得像一个标准的 Python :class:`file` 对象，但是它同样具有 
:meth:`~werkzeug.datastructures.FileStorage.save` 方法，该方法允许你存储文件在服务器的文件系统上。
这儿是一个简单的例子展示如何工作的::

    from flask import request

    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['the_file']
            f.save('/var/www/uploads/uploaded_file.txt')
        ...

如果你想要知道在上传到你的应用之前在客户端的文件名称，你可以访问 :attr:`~werkzeug.datastructures.FileStorage.filename` 
属性。但请记住永远不要信任这个值，因为这个值可以伪造。如果你想要使用客户端的文件名来在服务器上存储文件，
把它传递到 Werkzeug 提供给你的 :func:`~werkzeug.utils.secure_filename` 函数::

    from flask import request
    from werkzeug import secure_filename

    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['the_file']
            f.save('/var/www/uploads/' + secure_filename(f.filename))
        ...

一些更好的例子，请查看 :ref:`uploading-files` 。

Cookies
```````

你可以用 :attr:`~flask.Request.cookies` 属性来访问 cookies。你能够用响应对象的 :attr:`~flask.Response.set_cookie`
来设置 cookies。请求对象中的 :attr:`~flask.Request.cookies` 属性是一个客户端发送所有的 cookies 的字典。
如果你要使用会话(sessions)，请不要直接使用 cookies 相反用 Flask 中的 :ref:`sessions`，Flask 已经在 cookies 上增加了一些安全细节。

读取 cookies::

    from flask import request

    @app.route('/')
    def index():
        username = request.cookies.get('username')
        # use cookies.get(key) instead of cookies[key] to not get a
        # KeyError if the cookie is missing.

存储 cookies::

    from flask import make_response

    @app.route('/')
    def index():
        resp = make_response(render_template(...))
        resp.set_cookie('username', 'the username')
        return resp

注意 cookies 是在响应对象中被设置。由于通常只是从视图函数返回字符串， Flask 会将其转换为响应对象。
如果你要显式地这么做，你可以使用 :meth:`~flask.make_response` 函数接着修改它。

有时候你可能要在响应对象不存在的地方设置 cookie。利用 :ref:`deferred-callbacks` 模式使得这种情况成为可能。

为此也可以参阅 :ref:`about-responses`。

重定向和错误
--------------------

你能够用 :func:`~flask.redirect` 函数重定向用户到其它地方。能够用 :func:`~flask.abort` 函数提前中断一个请求并带有一个错误代码。
这里是一个它们如何工作的例子::

    from flask import abort, redirect, url_for

    @app.route('/')
    def index():
        return redirect(url_for('login'))

    @app.route('/login')
    def login():
        abort(401)
        this_is_never_executed()

这是一个相当无意义的例子因为用户会从主页重定向到一个不能访问的页面（ 401意味着禁止访问），
但是它说明了重定向如何工作。

默认情况下，每个错误代码会显示一个黑白错误页面。如果你想定制错误页面，可以使用 :meth:`~flask.Flask.errorhandler` 装饰器::

    from flask import render_template

    @app.errorhandler(404)
    def page_not_found(error):
        return render_template('page_not_found.html'), 404

注意到 ``404`` 是在 :func:`~flask.render_template` 调用之后。告诉 Flask 该页的错误代码应是 404 ，
即没有找到。默认的 200 被假定为：一切正常。

.. _about-responses:

关于响应
---------------

一个视图函数的返回值会被自动转换为一个响应对象。如果返回值是一个字符串，它被转换成一个响应主体是该字符串，错误代码为 ``200 OK`` ，媒体类型为 ``text/html`` 的响应对象。
Flask 把返回值转换成响应对象的逻辑如下：

1.  如果返回的是一个合法的响应对象，它会被从视图直接返回。
2.  如果返回的是一个字符串，响应对象会用字符串数据和默认参数创建。
3.  如果返回的是一个元组而且元组中元素能够提供额外的信息。这样的元组必须是 ``(response, status,
    headers)`` 形式且至少含有一个元素。 `status` 值将会覆盖状态代码，`headers` 可以是一个列表或额外的消息头值字典。
4.  如果上述条件均不满足，Flask 会假设返回值是一个合法的 WSGI 应用程序，并转换为一个请求对象。

如果你想要获取在视图中得到的响应对象，你可以用函数 :func:`~flask.make_response` 。

想象你有这样一个视图:

.. sourcecode:: python

    @app.errorhandler(404)
    def not_found(error):
        return render_template('error.html'), 404

你只需要用 :func:`~flask.make_response` 封装返回表达式，获取结果对象并修改，然后返回它：

.. sourcecode:: python

    @app.errorhandler(404)
    def not_found(error):
        resp = make_response(render_template('error.html'), 404)
        resp.headers['X-Something'] = 'A value'
        return resp

.. _sessions:

会话
--------

除了请求对象，还有第二个称为 :class:`~flask.session` 对象允许你在不同请求间存储特定用户的信息。
这是在 cookies 的基础上实现的，并且在 cookies 中使用加密的签名。这意味着用户可以查看 cookie 的内容，
但是不能修改它，除非它知道签名的密钥。

要使用会话，你需要设置一个密钥。这里介绍会话如何工作::

    from flask import Flask, session, redirect, url_for, escape, request

    app = Flask(__name__)

    @app.route('/')
    def index():
        if 'username' in session:
            return 'Logged in as %s' % escape(session['username'])
        return 'You are not logged in'

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        if request.method == 'POST':
            session['username'] = request.form['username']
            return redirect(url_for('index'))
        return '''
            <form action="" method="post">
                <p><input type=text name=username>
                <p><input type=submit value=Login>
            </form>
        '''

    @app.route('/logout')
    def logout():
        # remove the username from the session if it's there
        session.pop('username', None)
        return redirect(url_for('index'))

    # set the secret key.  keep this really secret:
    app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'

这里提到的 :func:`~flask.escape` 可以在你不使用模板引擎的时候做转义（如同本例）。

.. admonition:: 怎样产生一个好的密钥

   随机的问题在于很难判断什么是真随机。一个密钥应该足够随机。你的操作系统可以基于一个密码随机生成器来生成漂亮的随机值，这个值可以用来做密钥:

   >>> import os
   >>> os.urandom(24)
   '\xfd{H\xe5<\x95\xf9\xe3\x96.5\xd1\x01O<!\xd5\xa2\xa0\x9fR"\xa1\xa8'

   把这个值复制粘贴到你的代码，你就搞定了密钥。

使用基于 cookie 的会话需注意: Flask 会将你放进会话对象的值序列化到 cookie。如果你试图寻找一个跨请求不能存留的值， 
cookies 确实是启用的，并且你不会获得明确的错误信息，检查你页面请求中 cookie 的大小，并与 web 浏览器所支持的大小对比。


消息闪烁
----------------

好的应用和用户界面全部是关于反馈。如果用户得不到足够的反馈，他们可能会变得讨厌这个应用。Flask 提供了一个
真正的简单的方式来通过消息闪现系统给用户反馈。消息闪现系统基本上使得在请求结束时记录信息并在下一个
（且仅在下一个）请求中访问。通常结合模板布局来显示消息。

使用 :func:`~flask.flash` 方法来闪现一个消息，使用 :func:`~flask.get_flashed_messages` 能够获取消息，
:func:`~flask.get_flashed_messages` 也能用于模版中。针对一个完整的例子请查阅 :ref:`message-flashing-pattern`。

日志
-------

.. versionadded:: 0.3

有时候你会处于一种你处理的数据应该是正确的，然而实际上并不正确的状况。比如你可能有一些客户端代码，
代码向服务器发送一个 HTTP 请求但是显然它是畸形的。这可能是由于用户篡改数据，或客户端代码失败。
大部分时候针对这一情况返回 ``400 Bad Request`` 就可以了，但是有时候不行因为代码必须继续工作。

你可能仍然想要记录发生什么不正常事情。这时候日志就派上用处。从 Flask 0.3 开始日志记录是预先配置好的。

这里有一些日志调用的例子::

    app.logger.debug('A value for debugging')
    app.logger.warning('A warning occurred (%d apples)', 42)
    app.logger.error('An error occurred')

附带的 :attr:`~flask.Flask.logger` 是一个标准的日志类 :class:`~logging.Logger` ，因此更多的信息请
查阅官方文档 `logging
documentation <http://docs.python.org/library/logging.html>`_。

整合 WSGI 中间件
---------------------------

如果你想给你的应用添加 WSGI 中间件，你可以封装内部 WSGI 应用。例如如果你想使用 Werkzeug 包中的某个中间件来应付 lighttpd 中的 bugs，你可以这样做::

    from werkzeug.contrib.fixers import LighttpdCGIRootFix
    app.wsgi_app = LighttpdCGIRootFix(app.wsgi_app)

.. _quickstart_deployment:

部署到 Web 服务器
-------------------------

准备好部署你的新 Flask 应用？你可以立即部署到托管平台来完成快速入门，以下是向小项目提供免费的方案:

- `Deploying Flask on Heroku <http://devcenter.heroku.com/articles/python>`_
- `Deploying WSGI on dotCloud <http://docs.dotcloud.com/services/python/>`_
  with `Flask-specific notes <http://flask.pocoo.org/snippets/48/>`_

你可以托管 Flask 应用的其它选择：

- `Deploying Flask on Webfaction <http://flask.pocoo.org/snippets/65/>`_
- `Deploying Flask on Google App Engine <https://github.com/kamalgill/flask-appengine-template>`_
- `Sharing your Localhost Server with Localtunnel <http://flask.pocoo.org/snippets/89/>`_

如果你管理你自己的主机并且想要自己运行，请参看 :ref:`deployment`。
