.. _testing:

测试 Flask 应用
==========================

   **未经测试的东西是不完整的**

这句话的起源是未知的而且不完全正确，但是它离真理也不远了。未经测试的应用程序使其难以改善
现有的代码和未经测试的应用程序的开发人员往往会变得相当偏执。如果应用程序有自动测试，
您可以放心地进行更改并立即知道是否有任何问题。

Flask 提供了一种测试你的应用的方式，它通过使得 Werkzeug 测试 :class:`~werkzeug.test.Client` 暴露以及为你处理本地上下文。
然后你可以使用你最喜爱的测试解决方案。在本文档中我们将使用预装在 Python 的 :mod:`unittest` 包。

应用程序
---------------

首先，我们需要一个被测的应用；我们将采用来自 :ref:`tutorial` 的应用程序。如果你还没有这个应用程序，请从 
`the examples`_  获取源代码。

.. _the examples:
   http://github.com/mitsuhiko/flask/tree/master/examples/flaskr/

测试骨架
--------------------

为了测试应用，我们添加第二个模块( `flaskr_tests.py` )以及在里面创建一个 unittest 骨架::

    import os
    import flaskr
    import unittest
    import tempfile

    class FlaskrTestCase(unittest.TestCase):

        def setUp(self):
            self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
            flaskr.app.config['TESTING'] = True
            self.app = flaskr.app.test_client()
            flaskr.init_db()

        def tearDown(self):
            os.close(self.db_fd)
            os.unlink(flaskr.app.config['DATABASE'])

    if __name__ == '__main__':
        unittest.main()

在 :meth:`~unittest.TestCase.setUp` 方法中的代码创建了一个新的测试客户端并且初始化一个新的数据库。
这个函数在每个单独的测试函数运行之前被调用。为了在测试后删除数据库，我们在 :meth:`~unittest.TestCase.tearDown` 方法
中关闭了这个文件并且从文件系统中删除了它。此外在初始化过程中 ``TESTING`` 配置标志被激活。这将会使得处理请求时的错误捕捉失效，以便于 您在进行对应用发出请求的测试时获得更好的错误反馈。

这个测试客户端将提供给我们一个通向应用的简单接口，我们可以触发向应用发送请求的测试，并且此客户端也会为我们跟踪 cookies。

因为 SQLite 3 是基于文件系统的，我们可以轻易使用 tempfile 模块来创建一个临时数据库并且初始化它。:func:`~tempfile.mkstemp` 函数为我们做了两件事情：它返回一个低级别的文件句柄和一个随机文件名 ​​，我们使用后者作为数据库名。我们只需保持 `db_fd` 以便
我们能用 :func:`os.close` 函数来关闭这个文件。

如果我们现在运行测试套件，我们将会看到如下的输出::

    $ python flaskr_tests.py

    ----------------------------------------------------------------------
    Ran 0 tests in 0.000s

    OK

即使没有运行任何实际的测试，我们已经知道我们的 flaskr 应用语法上是有效的，否则在导入的时候就会抛出异常中断。

第一个测试
--------------

现在是时候开始测试应用程序的功能。让我们来检查应用程序是否显示 "No entries here so far"，当我们访问应用程序的根目录(``/``) 。
要做到这一点，我们在类中添加一个新的测试方法，像这样::

    class FlaskrTestCase(unittest.TestCase):

        def setUp(self):
            self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
            self.app = flaskr.app.test_client()
            flaskr.init_db()

        def tearDown(self):
            os.close(self.db_fd)
            os.unlink(flaskr.DATABASE)

        def test_empty_db(self):
            rv = self.app.get('/')
            assert 'No entries here so far' in rv.data

注意到我们的测试函数以 `test` 开头；这使得 :mod:`unittest` 能够自动识别要运行的测试方法。

通过使用 `self.app.get`，我们可以发送一个HTTP `GET` 请求到一个给定路径的应用程序。返回的值将会是一个
:class:`~flask.Flask.response_class` 对象。我们现在可以用 :attr:`~werkzeug.wrappers.BaseResponse.data` 属性
检查从应用中返回的值（作为字符串）。在这种情况下，我们将确保 ``'No entries here so far'`` 是输出的一部分。

再次运行它，你应该看到一个通过测试::

    $ python flaskr_tests.py
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.034s

    OK

登录和注销
------------------

我们应用的大部分功能只允许具有管理员资格的用户访问。所以我们需要一种方法来帮助我们的测试客户端登录和注销。
为此，我们向登录和注销页面发送一些请求，这些请求都携带了表单数据（用户名和密码），
因为登录和注销页面都会重定向，我们将客户端设置为 `follow_redirects` 。

添加如下的两种方法到你的 `FlaskrTestCase` 类::

   def login(self, username, password):
       return self.app.post('/login', data=dict(
           username=username,
           password=password
       ), follow_redirects=True)

   def logout(self):
       return self.app.get('/logout', follow_redirects=True)

现在我们可以轻易地测试正常地登录和注销以及因无效的认证而失败地登录。添加这个新的测试到类中::

   def test_login_logout(self):
       rv = self.login('admin', 'default')
       assert 'You were logged in' in rv.data
       rv = self.logout()
       assert 'You were logged out' in rv.data
       rv = self.login('adminx', 'default')
       assert 'Invalid username' in rv.data
       rv = self.login('admin', 'defaultx')
       assert 'Invalid password' in rv.data

测试添加消息
--------------------

我们应该测试添加消息是否正常，添加一个新的测试方法像这样::

    def test_messages(self):
        self.login('admin', 'default')
        rv = self.app.post('/add', data=dict(
            title='<Hello>',
            text='<strong>HTML</strong> allowed here'
        ), follow_redirects=True)
        assert 'No entries here so far' not in rv.data
        assert '&lt;Hello&gt;' in rv.data
        assert '<strong>HTML</strong> allowed here' in rv.data

这里我们检查 HTML 允许在正文但是不允许在标题，这是预期的行为。

运行这个测试，我们应该得到三个通过的测试:::

    $ python flaskr_tests.py
    ...
    ----------------------------------------------------------------------
    Ran 3 tests in 0.332s

    OK

关于请求头以及状态码更多复杂的测试，请查看 `MiniTwit Example`_ 源代码，它包含一个大型的测试套件。


.. _MiniTwit Example:
   http://github.com/mitsuhiko/flask/tree/master/examples/minitwit/


其它测试技巧
--------------------

除了如上文演示的使用测试客户端完成测试的方法，还有一个 :meth:`~flask.Flask.test_request_context` 方法可以用于配合 `with` 声明，用于触发一个临时的请求上下文。通过它，您可以访问 :class:`~flask.request` ，:class:`~flask.g` 和 :class:`~flask.session` 类的对象，就像在视图中一样。这里有一个完整的例子示范了这种用法::

    app = flask.Flask(__name__)

    with app.test_request_context('/?name=Peter'):
        assert flask.request.path == '/'
        assert flask.request.args['name'] == 'Peter'

以同样的方式，可以使用所有其它上下文绑定的对象。

如果你想测试在不同的配置下你的应用程序，这里似乎没有一个很好的方法，考虑切换到应用工厂 (请看 :ref:`app-factories`)。

值得注意的是如果你使用的是一个测试请求上下文，:meth:`~flask.Flask.before_request` 同 :meth:`~flask.Flask.after_request` 一样不会自动地被调用。然而 :meth:`~flask.Flask.teardown_request` 函数在测试请求的上下文离开with块确实会被执行。如果你
要 :meth:`~flask.Flask.before_request` 函数同样被调用，你必须自己调用 :meth:`~flask.Flask.preprocess_request`::

    app = flask.Flask(__name__)

    with app.test_request_context('/?name=Peter'):
        app.preprocess_request()
        ...

这对于打开数据库连接或者其他类似的操作来说，很可能是必须的，这视您应用的设计方式而定。

如果您希望调用 :meth:`~flask.Flask.after_request` 函数， 您需要使用 :meth:`~flask.Flask.process_response` 方法。 这个方法需要您传入一个 response 对象::

    app = flask.Flask(__name__)

    with app.test_request_context('/?name=Peter'):
        resp = Response('...')
        resp = app.process_response(resp)
        ...

这一般用处不大，因为这时候你可以直接地开始使用测试客户端。


伪造资源和上下文
----------------------------

.. versionadded:: 0.10

一个非常普遍的模式就是在用于的上下文或者 :attr:`flask.g` 对象中存储用户的认证信息以及数据库连接。
一般的模式是第一次使用的时候把它存入对象，然后在关闭的时候将其删除。想象下获取当前用户的代码::

    def get_user():
        user = getattr(g, 'user', None)
        if user is None:
            user = fetch_current_user_from_database()
            g.user = user
        return user

对于测试而言，不需要改变代码从外面覆盖用户将是很好的。这能够通过获取 :data:`flask.appcontext_pushed` 
信号来完成::

    from contextlib import contextmanager
    from flask import appcontext_pushed

    @contextmanager
    def user_set(app, user):
        def handler(sender, **kwargs):
            g.user = user
        with appcontext_pushed.connected_to(handler, app):
            yield

然后使用它::

    from flask import json, jsonify

    @app.route('/users/me')
    def users_me():
        return jsonify(username=g.user.username)

    with user_set(app, my_user):
        with app.test_client() as c:
            resp = c.get('/users/me')
            data = json.loads(resp.data)
            self.assert_equal(data['username'], my_user.username)


保持上下文
--------------------------

.. versionadded:: 0.4

有时触发一个通常的请求，但是将保持当前的上下文更长的时间，以便于附加的内省发生是很有用的。
在 Flask 0.4 中，在 `with` 块中使用 :meth:`~flask.Flask.test_client` 成为可能::

    app = flask.Flask(__name__)

    with app.test_client() as c:
        rv = c.get('/?tequila=42')
        assert request.args['tequila'] == '42'

如果你想要不在 `with` 块中使用 :meth:`~flask.Flask.test_client`，`assert` 将会失败因为 `request` 不再可用
(因为您试图在非真正请求中时候访问它)。然而，请记住任何 :meth:`~flask.Flask.after_request` 函数此时都已经 被执行了，所以您的数据库和一切相关的东西都可能已经被关闭。


访问和修改会话
--------------------------------

.. versionadded:: 0.8

有时候从测试客户端访问或者修改会话是十分有用的。通常这有两种方法实现。如果你只要确保一个会话拥有设置特定值的特定的键，
你只要保持上下文以及访问 :data:`flask.session`::

    with app.test_client() as c:
        rv = c.get('/')
        assert flask.session['foo'] == 42

然而这并不能使它可能还可以修改会话或访问会话在发送请求之前。从 Flask 0.8 开始，
我们提供一个叫做 ” Session 事务“ 的东西用于模拟适当的调用，从而在测试客户端的上下文中打开一个 Session，
并用于修改。在事务的结尾，Session 将被恢复为原来的样子。这些都独立于 Session 的后端使用::

    with app.test_client() as c:
        with c.session_transaction() as sess:
            sess['a_key'] = 'a value'

        # once this is reached the session was stored

值得注意地是在此时您必须使用 ``sess`` 对象而不是调用 :data:`flask.session` 代理，而这个对象本身提供了同样的接口。
