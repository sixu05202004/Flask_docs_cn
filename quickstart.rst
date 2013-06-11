.. _quickstart:

快速入门
==========

迫切希望上手？本文提供了一个很好的Flask介绍。假设你已经安装Flask，
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

把它保存成 `hello.py` (或者类似的文件)，然后用Python解释器运行它。确保你的应用不叫做 `flask.py`，
因为这会与Flask本身冲突。

::

    $ python hello.py
     * Running on http://127.0.0.1:5000/

现在浏览 `http://127.0.0.1:5000/ <http://127.0.0.1:5000/>`_，你会看到你的hello world问候。

那么这段代码做了什么？

1. 首先我们导入了类 :class:`~flask.Flask` 。这个类的实例化将会是我们的WSGI应用。第一个参数是应用模块的名称。
   如果你使用的是单一的模块（就如本例），第一个参数应该使用 `__name__`。因为取决于如果它以单独应用启动或作为模块导入，
   名称将会不同 （ ``'__main__'`` 对应于实际导入的名称）。获取更多的信息，请阅读 :class:`~flask.Flask` 的文档。
2. 接着，我们创建一个该类的实例。我们传递给它模块或包的名称。这样Flask才会知道去哪里寻找模板、静态文件等等。
3. 我们使用装饰器 :meth:`~flask.Flask.route` 告诉Flask哪个URL才能触发我们的函数。
4. 定义一个函数，该函数名也是用来给特定函数生成URLs，并且返回我们想要显示在用户浏览器上的信息。
5. 最后我们用函数 :meth:`~flask.Flask.run` 启动本地服务器来运行我们的应用。``if __name__ == '__main__':``
   确保服务器只会在该脚本被Python解释器直接执行的时候才会运行，而不是作为模块导入的时候。

请按 control-C 来停止服务器。

.. _public-server:

.. admonition:: 外部可见服务器 

   当你运行服务器，你会注意到它只能从你自己的计算机上访问，网络中其它任何的地方都不能访问。
   这是因为默认情况下，调试模式，应用中的一个用户可以执行你计算机上的任意Python代码。

   如果你关闭 `debug` 或者信任你所在网络上的用户，你可以让你的服务器对外可用，只要简单地改变方法
   :meth:`~flask.Flask.run` 的调用像如下这样::

       app.run(host='0.0.0.0')

   这让你的操作系统去监听所有公开的IP。


.. _debug-mode:

调试模式
----------

:meth:`~flask.Flask.run` 方法是十分适用于启动一个本地开发服务器，但是你需要在修改代码后手动重启服务器。
这样做并不好，Flask能做得更好。如果启用了调试支持，在代码修改的时候服务器能够自动加载，
并且如果发生错误，它会提供一个有用的调试器。

有两种方式开启调式模式。一种是在应用对象上设置标志位::

    app.debug = True
    app.run()

或者作为run的一个参数传入::

    app.run(debug=True)

两种方法效果是一样的。

.. admonition:: Attention

   尽管交互式调试器不能在分叉(forking)环境上工作(这使得它几乎不可能在生产服务器上使用)，
   它依然允许执行任意代码。这使它成为一个巨大的安全风险，因此它 **绝对不能用于生成环境**。

运行中的调试器的截图:

.. image:: _static/debugger.png
   :align: center
   :class: screenshot
   :alt: screenshot of debugger in action

是不是还有其它的调试器？请查看 :ref:`working-with-debuggers`。


路由
-------

现代Web应用程序有优雅的URLs。这能够帮助人们记住URLs，这点在面向使用慢网络连接的移动设备的应用上有用。
如果用户不必通过点击首页而直接访问想要的页面，很可能他们会喜欢这个页面而且下次再次访问。

正如上面所说，meth:`~flask.Flask.route` 装饰器是用于把一个函数绑定到一个URL上。这有些基本的例子::

    @app.route('/')
    def index():
        return 'Index Page'

    @app.route('/hello')
    def hello():
        return 'Hello World'

但是不仅如此！你可以动态地构造URL的特定部分，也可以在一个函数上附加多个规则。

变量规则
``````````````

为了给URL增加变量的部分，你需要把一些特定的字段标记成 ``<variable_name>``。这些特定的字段
将作为参数传入到你的函数中。当然也可以指定一个可选的转换器通过规则 ``<converter:variable_name>``。
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

   Flask的URL规则是基于Werkzeug的routing模块。
   该模块背后的想法是基于Apache和早期的HTTP服务器定下先例确保美丽和唯一的URL。  

   以这两个规则为例::

        @app.route('/projects/')
        def projects():
            return 'The project page'

        @app.route('/about')
        def about():
            return 'The about page'

   虽然它们看起来确实相似，但它们结尾斜线的使用在URL *定义* 中不同。 
   第一种情况中，规范的URL指向projects尾端有一个斜线。
   这种感觉很像在文件系统中的文件夹。访问一个结尾不带斜线的URL会被Flask重定向到带斜线的规范URL去。

   然而，第二种情况的URL结尾不带斜线，类似UNIX-like系统下的文件的路径名。
   访问结尾带斜线的URL会产生一个 404 “Not Found” 错误。

   当用户访问页面时忘记结尾斜线时，这个行为允许关联的URL继续工作，
   并且与Apache和其它的服务器的行为一致。另外，URL会保持唯一，有助于避免搜索引擎索引同一个页面两次。


.. _url-building:

构建URL
````````````

如果它可以匹配URL，那么Flask能够生成它们吗？当然Flask能够做到。你可以使用函数
:func:`~flask.url_for` 来针对一个特定的函数构建一个URL。它能够接受函数名作为第一参数，以及一些关键字参数，
每一个关键字参数对应于URL规则的变量部分。未知变量部分被插入到URL中作为查询参数。这里有些例子：

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

(这里也使用了 :meth:`~flask.Flask.test_request_context` 方法，下面会给出解释。这个方法告诉Flask表现得像是在处理一个请求，即使我们正在通过Python的shell交互。
请看下面的解释。 :ref:`context-locals`)。

为什么你愿意构建URLs而不是在模版中硬编码？这里有三个好的理由：

1. 反向构建通常比硬编码更具备描述性。更重要的是，它允许你一次性修改URL， 
   而不是到处找URL修改。
2. 构建URL能够显式地处理特殊字符和Unicode转义，因此你不必去处理这些。
3. 如果你的应用不在URL根目录下(比如，在
   ``/myapplication`` 而不在 ``/``)， :func:`~flask.url_for` 将会适当地替你处理好。


HTTP方法
````````````

HTTP (也就说web应用协议)有不同的方法来访问URLs。默认情况下，路由只会响应 `GET` 请求，
但是能够通过给 :meth:`~flask.Flask.route` 装饰器提供 `methods` 参数来改变。这里是些例子::

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        if request.method == 'POST':
            do_the_login()
        else:
            show_the_login_form()

如果使用 `GET` 方法，`HEAD`方法 将会自动添加进来。你不必处理它们。也能确保 `HEAD` 请求
会按照 `HTTP RFC`_ (文档在HTTP协议里面描述) 要求来处理， 因此你完全可以忽略这部分HTTP规范。
同样地，自从Flask0.6后，`OPTIONS`也能自动为你处理。

也许你并不清楚HTTP方法是什么？别担心，这里有一个HTTP方法的快速入门以及为什么它们重要：

HTTP方法（通常也称为“谓词”）告诉服务器客户端想要对请求的页面 *做* 什么。下面这些方法是比较常见的：

`GET`
    浏览器通知服务器只 *获取* 页面上的信息并且发送回来。这可能是最常用的方法。 

`HEAD`
    浏览器告诉服务器获取信息，但是只对 *头信息* 感兴趣，不需要整个页面的内容。
    应用应该处理起来像接收到一个 `GET` 请求但是不传递实际内容。在Flask中你完全不需要处理它，
    底层的Werkzeug库会为你处理的。

`POST`
    浏览器通知服务器它要在URL上 *提交*一些信息，服务器必须保证数据被存储且只存储一次。
    这是HTML表单通常发送数据到服务器的方法。

`PUT`
    同 `POST` 类似，但是服务器可能触发了多次存储过程，多次覆盖掉旧值。现在你就会问这有什么用，
    有许多理由需要如此去做。考虑下在传输过程中连接丢失：在这种情况下浏览器 和服务器之间的系统可能安全地第二次接收请求，而不破坏其它东西。对于 `POST` 是不可能实现的，因为
    它只会被触发一次。 

`DELETE`
    移除给定位置的信息。

`OPTIONS`
    给客户端提供一个快速的途径来指出这个URL支持哪些HTTP方法。从Flask 0.6开始，自动实现了它。

现在比较有兴趣的是在HTML4和XHTML1，表单只能以 `GET` 和 `POST` 方法来提交到服务器。在JavaScript和以后的
HTML标准中也能使用其它的方法。同时，HTTP最近变得十分流行，浏览器不再是唯一使用HTTP的客户端。比如，许多
版本控制系统使用HTTP。 

.. _HTTP RFC: http://www.ietf.org/rfc/rfc2068.txt

静态文件
------------

动态的web应用同样需要静态文件。CSS和JavaScript文件通常来源于此。理想情况下，
你的web服务器已经配置好为它们服务，然而在开发过程中Flask能够做到。
只要在你的包中或模块旁边创建一个名为 `static` 的文件夹，在应用中使用 `/static` 即可访问。

给静态文件生成 URL ，使用特殊的 ``'static'`` 端点名::

    url_for('static', filename='style.css')

这个文件应该存储在文件系统上称为 ``static/style.css``。

渲染模板
-------------------

在Python中生成HTML并不好玩，实际上是相当繁琐的，因为你必须自行做好HTML转义以保持应用程序的安全。
由于这个原因，Flask自动为你配置好 `Jinja2
<http://jinja.pocoo.org/2/>`_ 模版。

你可以使用方法 :func:`~flask.render_template` 来渲染模版。所有你需要做的就是提供模版的名称以及
你想要作为关键字参数传入模板的变量。这里有个渲染模版的简单例子::

    from flask import render_template

    @app.route('/hello/')
    @app.route('/hello/<name>')
    def hello(name=None):
        return render_template('hello.html', name=name)

Flask将会在 `templates` 文件夹中寻找模版。因此如果你的应用是个模块，这个文件 夹在模块的旁边，如果它是一个包，那么这个文件夹在你的包里面:

**Case 1**: a module::

    /application.py
    /templates
        /hello.html

**Case 2**: a package::

    /application
        /__init__.py
        /templates
            /hello.html

对于模板，你可以使用Jinja2模板的全部能力。详细信息查看官方的 `Jinja2 Template Documentation
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

自动转义是开启的，因此如果 `name` 包含HTML，它将会自动转义。如果你信任一个变量，并且你知道它是安全的
（例如一个模块把wiki标记转换到HTML），你可以用 :class:`~jinja2.Markup` 类或 ``|safe`` 过滤器在模板中标记它是安全的。
在Jinja 2文档中，你会见到更多例子。

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

For web applications it's crucial to react to the data a client sent to
the server.  In Flask this information is provided by the global
:class:`~flask.request` object.  If you have some experience with Python
you might be wondering how that object can be global and how Flask
manages to still be threadsafe.  The answer is context locals:


.. _context-locals:

上下文
``````````````

.. admonition:: Insider Information

   If you want to understand how that works and how you can implement
   tests with context locals, read this section, otherwise just skip it.

Certain objects in Flask are global objects, but not of the usual kind.
These objects are actually proxies to objects that are local to a specific
context.  What a mouthful.  But that is actually quite easy to understand.

Imagine the context being the handling thread.  A request comes in and the
web server decides to spawn a new thread (or something else, the
underlying object is capable of dealing with concurrency systems other
than threads).  When Flask starts its internal request handling it
figures out that the current thread is the active context and binds the
current application and the WSGI environments to that context (thread).
It does that in an intelligent way so that one application can invoke another
application without breaking.

So what does this mean to you?  Basically you can completely ignore that
this is the case unless you are doing something like unit testing.  You
will notice that code which depends on a request object will suddenly break
because there is no request object.  The solution is creating a request
object yourself and binding it to the context.  The easiest solution for
unit testing is to use the :meth:`~flask.Flask.test_request_context`
context manager.  In combination with the `with` statement it will bind a
test request so that you can interact with it.  Here is an example::

    from flask import request

    with app.test_request_context('/hello', method='POST'):
        # now you can do something with the request until the
        # end of the with block, such as basic assertions:
        assert request.path == '/hello'
        assert request.method == 'POST'

The other possibility is passing a whole WSGI environment to the
:meth:`~flask.Flask.request_context` method::

    from flask import request

    with app.request_context(environ):
        assert request.method == 'POST'

请求对象
``````````````````

The request object is documented in the API section and we will not cover
it here in detail (see :class:`~flask.request`). Here is a broad overview of
some of the most common operations.  First of all you have to import it from
the `flask` module::

    from flask import request

The current request method is available by using the
:attr:`~flask.request.method` attribute.  To access form data (data
transmitted in a `POST` or `PUT` request) you can use the
:attr:`~flask.request.form` attribute.  Here is a full example of the two
attributes mentioned above::

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

What happens if the key does not exist in the `form` attribute?  In that
case a special :exc:`KeyError` is raised.  You can catch it like a
standard :exc:`KeyError` but if you don't do that, a HTTP 400 Bad Request
error page is shown instead.  So for many situations you don't have to
deal with that problem.

To access parameters submitted in the URL (``?key=value``) you can use the
:attr:`~flask.request.args` attribute::

    searchword = request.args.get('key', '')

We recommend accessing URL parameters with `get` or by catching the
`KeyError` because users might change the URL and presenting them a 400
bad request page in that case is not user friendly.

For a full list of methods and attributes of the request object, head over
to the :class:`~flask.request` documentation.


文件上传
````````````

You can handle uploaded files with Flask easily.  Just make sure not to
forget to set the ``enctype="multipart/form-data"`` attribute on your HTML
form, otherwise the browser will not transmit your files at all.

Uploaded files are stored in memory or at a temporary location on the
filesystem.  You can access those files by looking at the
:attr:`~flask.request.files` attribute on the request object.  Each
uploaded file is stored in that dictionary.  It behaves just like a
standard Python :class:`file` object, but it also has a
:meth:`~werkzeug.datastructures.FileStorage.save` method that allows you to store that
file on the filesystem of the server.  Here is a simple example showing how
that works::

    from flask import request

    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['the_file']
            f.save('/var/www/uploads/uploaded_file.txt')
        ...

If you want to know how the file was named on the client before it was
uploaded to your application, you can access the
:attr:`~werkzeug.datastructures.FileStorage.filename` attribute.  However please keep in
mind that this value can be forged so never ever trust that value.  If you
want to use the filename of the client to store the file on the server,
pass it through the :func:`~werkzeug.utils.secure_filename` function that
Werkzeug provides for you::

    from flask import request
    from werkzeug import secure_filename

    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['the_file']
            f.save('/var/www/uploads/' + secure_filename(f.filename))
        ...

For some better examples, checkout the :ref:`uploading-files` pattern.

Cookies
```````

To access cookies you can use the :attr:`~flask.Request.cookies`
attribute.  To set cookies you can use the
:attr:`~flask.Response.set_cookie` method of response objects.  The
:attr:`~flask.Request.cookies` attribute of request objects is a
dictionary with all the cookies the client transmits.  If you want to use
sessions, do not use the cookies directly but instead use the
:ref:`sessions` in Flask that add some security on top of cookies for you.

Reading cookies::

    from flask import request

    @app.route('/')
    def index():
        username = request.cookies.get('username')
        # use cookies.get(key) instead of cookies[key] to not get a
        # KeyError if the cookie is missing.

Storing cookies::

    from flask import make_response

    @app.route('/')
    def index():
        resp = make_response(render_template(...))
        resp.set_cookie('username', 'the username')
        return resp

Note that cookies are set on response objects.  Since you normally
just return strings from the view functions Flask will convert them into
response objects for you.  If you explicitly want to do that you can use
the :meth:`~flask.make_response` function and then modify it.

Sometimes you might want to set a cookie at a point where the response
object does not exist yet.  This is possible by utilizing the
:ref:`deferred-callbacks` pattern.

For this also see :ref:`about-responses`.

重定向和错误
--------------------

To redirect a user to somewhere else you can use the
:func:`~flask.redirect` function. To abort a request early with an error
code use the :func:`~flask.abort` function.  Here an example how this works::

    from flask import abort, redirect, url_for

    @app.route('/')
    def index():
        return redirect(url_for('login'))

    @app.route('/login')
    def login():
        abort(401)
        this_is_never_executed()

This is a rather pointless example because a user will be redirected from
the index to a page they cannot access (401 means access denied) but it
shows how that works.

By default a black and white error page is shown for each error code.  If
you want to customize the error page, you can use the
:meth:`~flask.Flask.errorhandler` decorator::

    from flask import render_template

    @app.errorhandler(404)
    def page_not_found(error):
        return render_template('page_not_found.html'), 404

Note the ``404`` after the :func:`~flask.render_template` call.  This
tells Flask that the status code of that page should be 404 which means
not found.  By default 200 is assumed which translates to: all went well.

.. _about-responses:

关于响应
---------------

The return value from a view function is automatically converted into a
response object for you.  If the return value is a string it's converted
into a response object with the string as response body, an ``200 OK``
error code and a ``text/html`` mimetype.  The logic that Flask applies to
converting return values into response objects is as follows:

1.  If a response object of the correct type is returned it's directly
    returned from the view.
2.  If it's a string, a response object is created with that data and the
    default parameters.
3.  If a tuple is returned the items in the tuple can provide extra
    information.  Such tuples have to be in the form ``(response, status,
    headers)`` where at least one item has to be in the tuple.  The
    `status` value will override the status code and `headers` can be a
    list or dictionary of additional header values.
4.  If none of that works, Flask will assume the return value is a
    valid WSGI application and convert that into a response object.

If you want to get hold of the resulting response object inside the view
you can use the :func:`~flask.make_response` function.

Imagine you have a view like this:

.. sourcecode:: python

    @app.errorhandler(404)
    def not_found(error):
        return render_template('error.html'), 404

You just need to wrap the return expression with
:func:`~flask.make_response` and get the result object to modify it, then
return it:

.. sourcecode:: python

    @app.errorhandler(404)
    def not_found(error):
        resp = make_response(render_template('error.html'), 404)
        resp.headers['X-Something'] = 'A value'
        return resp

.. _sessions:

会话
--------

In addition to the request object there is also a second object called
:class:`~flask.session` which allows you to store information specific to a
user from one request to the next.  This is implemented on top of cookies
for you and signs the cookies cryptographically.  What this means is that
the user could look at the contents of your cookie but not modify it,
unless they know the secret key used for signing.

In order to use sessions you have to set a secret key.  Here is how
sessions work::

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

The :func:`~flask.escape` mentioned here does escaping for you if you are
not using the template engine (as in this example).

.. admonition:: How to generate good secret keys

   The problem with random is that it's hard to judge what is truly random.  And
   a secret key should be as random as possible.  Your operating system
   has ways to generate pretty random stuff based on a cryptographic
   random generator which can be used to get such a key:

   >>> import os
   >>> os.urandom(24)
   '\xfd{H\xe5<\x95\xf9\xe3\x96.5\xd1\x01O<!\xd5\xa2\xa0\x9fR"\xa1\xa8'

   Just take that thing and copy/paste it into your code and you're done.

A note on cookie-based sessions: Flask will take the values you put into the
session object and serialize them into a cookie.  If you are finding some
values do not persist across requests, cookies are indeed enabled, and you are
not getting a clear error message, check the size of the cookie in your page
responses compared to the size supported by web browsers.


消息闪烁
----------------

Good applications and user interfaces are all about feedback.  If the user
does not get enough feedback they will probably end up hating the
application.  Flask provides a really simple way to give feedback to a
user with the flashing system.  The flashing system basically makes it
possible to record a message at the end of a request and access it on the next
(and only the next) request.  This is usually combined with a layout
template to expose the message.

To flash a message use the :func:`~flask.flash` method, to get hold of the
messages you can use :func:`~flask.get_flashed_messages` which is also
available in the templates.  Check out the :ref:`message-flashing-pattern`
for a full example.

日志
-------

.. versionadded:: 0.3

Sometimes you might be in a situation where you deal with data that
should be correct, but actually is not.  For example you may have some client-side
code that sends an HTTP request to the server but it's obviously
malformed.  This might be caused by a user tampering with the data, or the
client code failing.  Most of the time it's okay to reply with ``400 Bad
Request`` in that situation, but sometimes that won't do and the code has
to continue working.

You may still want to log that something fishy happened.  This is where
loggers come in handy.  As of Flask 0.3 a logger is preconfigured for you
to use.

Here are some example log calls::

    app.logger.debug('A value for debugging')
    app.logger.warning('A warning occurred (%d apples)', 42)
    app.logger.error('An error occurred')

The attached :attr:`~flask.Flask.logger` is a standard logging
:class:`~logging.Logger`, so head over to the official `logging
documentation <http://docs.python.org/library/logging.html>`_ for more
information.

Hooking in WSGI Middlewares
---------------------------

If you want to add a WSGI middleware to your application you can wrap the
internal WSGI application.  For example if you want to use one of the
middlewares from the Werkzeug package to work around bugs in lighttpd, you
can do it like this::

    from werkzeug.contrib.fixers import LighttpdCGIRootFix
    app.wsgi_app = LighttpdCGIRootFix(app.wsgi_app)

.. _quickstart_deployment:

部署到Web服务器
-------------------------

准备好部署你的新Flask应用？你可以立即部署到托管平台来完成快速入门，以下是向小项目提供免费的方案:

- `Deploying Flask on Heroku <http://devcenter.heroku.com/articles/python>`_
- `Deploying WSGI on dotCloud <http://docs.dotcloud.com/services/python/>`_
  with `Flask-specific notes <http://flask.pocoo.org/snippets/48/>`_

你可以托管Flask应用的其它选择：

- `Deploying Flask on Webfaction <http://flask.pocoo.org/snippets/65/>`_
- `Deploying Flask on Google App Engine <https://github.com/kamalgill/flask-appengine-template>`_
- `Sharing your Localhost Server with Localtunnel <http://flask.pocoo.org/snippets/89/>`_

如果你管理你自己的主机并且想要自己运行，请参看 :ref:`deployment`。
