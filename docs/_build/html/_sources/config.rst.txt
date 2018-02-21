.. _config:

配置处理
======================

.. versionadded:: 0.3

应用程序需要某种形式的配置。你可能会需要根据应用环境更改不同的设置，比如开关调试模式、
设置密钥、或是别的设定环境的东西。

Flask 被设计为需要配置来启动应用。你可以在代码中硬编码配置，这对于小的应用并不坏，但是有更好的方法。

跟你如何加载配置无关，有一个配置对象用来维持加载的配置值：:class:`~flask.Flask` 对象的 :attr:`~flask.Flask.config` 属性。
这是Flask自身放置特定配置的地方同时也是扩展放置它们配置值的地方。但是，这里也可以放置你自己的配置。

基本配置
--------------------

:attr:`~flask.Flask.config` 实际上是字典的一个子类且能够像字典一样被修改::

    app = Flask(__name__)
    app.config['DEBUG'] = True

某些配置也被传入到 :attr:`~flask.Flask` 对象因此你可以在那里读取它们::

    app.debug = True

你能够用 :meth:`dict.update` 方法一次性地更新多个键值::

    app.config.update(
        DEBUG=True,
        SECRET_KEY='...'
    )

内置的配置值
----------------------------

下列配置值是 Flask 内部使用的：

.. tabularcolumns:: |p{6.5cm}|p{8.5cm}|

================================= =============================================================
``DEBUG``                         启用/禁止调试模式
``TESTING``                       启用/禁止测试模式
``PROPAGATE_EXCEPTIONS``          显式地启用或者禁止异常的传播。
                                  如果没有设置 或显式地设置为 `None` ，
                                  当 `TESTING` 或 `DEBUG` 为真时，
                                  隐式为真。
``PRESERVE_CONTEXT_ON_EXCEPTION`` 默认情况下，如果应用工作在调试模式，
                                  请求上下文不会在异常时出栈来允许调试器内省。 
                                  这可以通过这个键来禁用。
                                  你同样可以用这个设定来强制启用它，
                                  即使没有调试执行，这对调试生产应用很有用
                                  (但风险也很大)
``SECRET_KEY``                    密钥
``SESSION_COOKIE_NAME``           会话 cookie 的名称
``SESSION_COOKIE_DOMAIN``         会话 cookie 的域。如果没有设置的话，
                                  cookie 将会对 ``SERVER_NAME`` 所有的子域都有效。
``SESSION_COOKIE_PATH``           会话 cookie 的路径。如果没有设置或者没有为 ``'/'`` 
                                  设置，cookie 将会对所有的 ``APPLICATION_ROOT`` 
                                  有效。
``SESSION_COOKIE_HTTPONLY``       控制 cookie 是否应被设置 httponly 的标志， 
                                  默认为 `True` 。
``SESSION_COOKIE_SECURE``         控制 cookie 是否应被设置安全标志，默认为 `False`。
``PERMANENT_SESSION_LIFETIME``    一个持久化的会话的生存时间，作为一个 
                                  :class:`datetime.timedelta` 对象。从 Flask0.8 开始
                                  也可以用一个整数来表示秒。
``USE_X_SENDFILE``                启用/禁止 x-sendfile
``LOGGER_NAME``                   日志记录器的名称
``SERVER_NAME``                   服务器的名称以及端口，需要它为了支持子域名
                                  (如: ``'myapp.dev:5000'``)。注意 localhost 是
                                  不支持子域名的因此设置成 “localhost” 是无意义的。
                                  设置 ``SERVER_NAME`` 默认会允许在没有请求上下文 
                                  而仅有应用上下文时生成 URL。
``APPLICATION_ROOT``              如果应用不占用完整的域名或子域名，
                                  这个选项可以被设置为应用所在的路径。
                                  这个路径也会用于会话 cookie 的路径值。
                                  如果直接使用域名，则留作 ``None``。 
``MAX_CONTENT_LENGTH``            如果设置为字节数， Flask 会拒绝内容长度大于
                                  此值的请求进入，并返回一个 413 状态码。
``SEND_FILE_MAX_AGE_DEFAULT``:    默认缓存控制的最大期限，以秒计，
                                  在 :meth:`~flask.Flask.send_static_file` 
                                  (默认的静态文件处理器)和 
                                  :func:`~flask.send_file` 中使用。 
                                  对于单个文件，覆盖这个值，使用
                                  :meth:`~flask.Flask.get_send_file_max_age` 勾住
                                  :class:`~flask.Flask` 或者 :class:`~flask.Blueprint`。
                                  默认为 43200（12小时）。 
``TRAP_HTTP_EXCEPTIONS``          如果这个值被设置为 ``True`` ，
                                  Flask 不会执行 HTTP 异常的错误处理，
                                  而是像对待其它异常一样，通过异常栈让它冒泡。
                                  这对于需要找出 HTTP 异常源头的可怕调试情形是有用的。
``TRAP_BAD_REQUEST_ERRORS``       Werkzeug 处理请求中的特定数据的内部数据结构会 
                                  抛出同样也是“错误的请求”异常的特殊的 key errors 。
                                  同样地，为了保持一致，许多操作可以 
                                  显式地抛出 BadRequest 异常。因为在调试中，
                                  你希望准确地找出异常的原因，
                                  这个设置用于在这些情形下调试。
                                  如果这个值被设置为 ``True`` ，
                                  你只会得到常规的回溯。
``PREFERRED_URL_SCHEME``          URL 模式用于 URL 生成。如果没有设置 URL 模式， 
                                  默认将为 ``http`` 。
``JSON_AS_ASCII``                 默认情况下 Flask 序列化对象成 ascii 编码的 JSON。
                                  如果不对该配置项就行设置的话，Flask 将不会编码成
                                  ASCII 保持字符串原样，并且返回 unicode 字符串。``jsonfiy``
                                  会自动按照 ``utf-8`` 进行编码并且传输。
``JSON_SORT_KEYS``                默认情况下 Flask 将会依键值顺序的方式序列化 JSON。
                                  这样做是为了确保字典哈希种子的独立性，返回值将会一致不会造成
                                  额外的 HTTP 缓存。通过改变这个变量可以重载默认行为。
                                  这是不推荐也许会带来缓存消耗的性能问题。
``JSONIFY_PRETTYPRINT_REGULAR``   如果设置成 ``True`` (默认下)，jsonify 响应将会完美地打印。
================================= =============================================================

.. admonition:: 更多关于 ``SERVER_NAME`` 内容

   ``SERVER_NAME`` 键是用于子域名支持。因为 Flask 在得知现有服务器名之前不能 猜测出子域名部分，所以如果你想使用子域名，这个选项必要的，并且也用于会话 cookie。

   请牢记不只有 Flask 存在不知道子域名的问题，你的浏览器同样存在这样的问题。
   大多数现代 web 浏览器不允许服务器名不含有点的跨子域名 cookie。因此如果你的服务器的
   名称为 ``'localhost'``，你将不能为 ``'localhost'`` 和所有它的子域名设置一个 cookie。
   请选择一个合适的服务器名，像 ``'myapplication.local'`` ， 并添加你想要的服务器名 + 子域名 
   到你的 host 配置或设置一个本地 `bind`_ 。

.. _bind: https://www.isc.org/software/bind

.. versionadded:: 0.4
   ``LOGGER_NAME``

.. versionadded:: 0.5
   ``SERVER_NAME``

.. versionadded:: 0.6
   ``MAX_CONTENT_LENGTH``

.. versionadded:: 0.7
   ``PROPAGATE_EXCEPTIONS``, ``PRESERVE_CONTEXT_ON_EXCEPTION``

.. versionadded:: 0.8
   ``TRAP_BAD_REQUEST_ERRORS``, ``TRAP_HTTP_EXCEPTIONS``,
   ``APPLICATION_ROOT``, ``SESSION_COOKIE_DOMAIN``,
   ``SESSION_COOKIE_PATH``, ``SESSION_COOKIE_HTTPONLY``,
   ``SESSION_COOKIE_SECURE``

.. versionadded:: 0.9
   ``PREFERRED_URL_SCHEME``

.. versionadded:: 0.10
   ``JSON_AS_ASCII``, ``JSON_SORT_KEYS``, ``JSONIFY_PRETTYPRINT_REGULAR``

从文件中配置
----------------------

如果你能在独立的文件里存储配置，理想情况是存储在实际的应用包之外，它将变得更有用。
这能够使得打包和分发你的应用程序通过不同的处理工具( :ref:`distribute-deployment` )，
之后才修改配置文件。

因此一个共同的模式是这样的::

    app = Flask(__name__)
    app.config.from_object('yourapplication.default_settings')
    app.config.from_envvar('YOURAPPLICATION_SETTINGS')

首先从 `yourapplication.default_settings` 模块加载配置，接着用 
:envvar:`YOURAPPLICATION_SETTINGS` 环境变量指向的文件的内容覆盖其值。在 Linux 或者 OS X 上，
这个环境变量可以在启动服务器之前，在 shell 上 export 命令设置::

    $ export YOURAPPLICATION_SETTINGS=/path/to/settings.cfg
    $ python run-app.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader...

在 Windows 系统上，使用内置的 `set` 代替::

    >set YOURAPPLICATION_SETTINGS=\path\to\settings.cfg

配置文件本身实际上是 Python 文件。只有大写名称的值才会被存储到配置对象中。
因此请确保在配置键中使用了大写字母。  

这里是一个配置文件的例子::

    # Example configuration
    DEBUG = False
    SECRET_KEY = '?\xbf,\xb4\x8d\xa3"<\x9c\xb0@\x0f5\xab,w\xee\x8d$0\x13\x8b83'

确保尽早地载入配置，这样扩展才能在启动时访问配置。还有其他方式从不同的文件中加载配置对象。
完整的介绍请查阅 :class:`~flask.Config` 对象的文档。


配置最佳实践
----------------------------

前面提到的方法的缺点是使测试有点困难。通常对于这个问题没有单一 100% 的解决方案，但是
你可以注意下面的事项来改善:

1.  在函数中创建你的应用，并在上面注册蓝图。这样你可以用不同的配置来创建多个应用实例，
    以此使得单元测试变得很简单。你可以用这样的方法来按需传入配置。
2.  不要写出在导入时需要配置的代码。如果你限制只在请求中访问配置，
    你可以在之后按需重新配置对象。


开发/生产
------------------------

大多数应用程序需要不止一个配置。至少对生产服务器和开发服务器有独立的配置。最容易的处理方式就是
使用一个总是被加载的默认配置和部分版本控制，以及一个独立的配置像上面例子提及到的覆盖必要的配置值::

    app = Flask(__name__)
    app.config.from_object('yourapplication.default_settings')
    app.config.from_envvar('YOURAPPLICATION_SETTINGS')

接着你只要新建一个独立的 `config.py` 文件并且导入 ``YOURAPPLICATION_SETTINGS=/path/to/config.py``。
不过也有替代方法。例如你可以使用导入或者子类化。

在 Django 世界中流行的是在文件顶部，显式地使用 ``from yourapplication.default_settings import *`` 导入配置文件，并手动覆盖更改。你也可以检查一个类似 ``YOURAPPLICATION_MODE`` 的环境变量来设置 
`production` ， `development` 等等，并导入基于此的不同的硬编码文件。

一个有趣的模式也是为配置使用类和继承::

    class Config(object):
        DEBUG = False
        TESTING = False
        DATABASE_URI = 'sqlite://:memory:'

    class ProductionConfig(Config):
        DATABASE_URI = 'mysql://user@localhost/foo'

    class DevelopmentConfig(Config):
        DEBUG = True

    class TestingConfig(Config):
        TESTING = True

为了使得这样一个配置有用你只要调用 :meth:`~flask.Config.from_object`::

    app.config.from_object('configmodule.ProductionConfig')

有许多不同处理配置文件方式，这取决于你想要如何管理配置文件。不过这里有一些好的建议：

-   在版本控制中保留一个默认配置。在覆盖配置值之前要么用默认的配置填充你的配置，
    要么在你的配置文件中导入它。
-   使用环境变量来在配置间切换。这样可以从 Python 解释器之外完成，使开发和部署更容易，
    因为你可以在不触及代码的情况下快速简便地切换配置。如果你经常在不同的项目中作业，
    你甚至可以创建激活一个 virtualenv 并导出开发 配置的脚本。
-   使用一个类似 `fabric`_ 工具在生成环境向生成服务器分别推送代码和配置。对于如何做到这一点的细节，
    请查阅 :ref:`fabric-deployment` 。

.. _fabric: http://fabfile.org/


.. _instance-folders:

示例文件夹
----------------

.. versionadded:: 0.8

Flask 0.8 引入了示例文件夹。Flask 在很长时间使得直接引用相对应用文件夹 的路径成为可能。这也是许多开发者加载存储在载入应用旁边的配置的方法。不幸 的是，这只会在应用不是包，即根路径指向包内容的情况下才能工作。

在 Flask 0.8 中，引入一个新的属性： :attr:`Flask.instance_path`。它涉及到一个新的称为
“示例文件夹”的概念。实例文件夹被为不使用版本控制和特定的部署而设计。
这是放置运行时更改的文件和配置文件的最佳位置。

创建 Flask 应用的时候你可以显式地提供示例文件夹路径或者让 Flask 自动识别实例文件夹。对于显式的配置，使用 `instance_path` 参数::

    app = Flask(__name__, instance_path='/path/to/instance/folder')

请注意给出的 *一定* 是绝对路径。

如果 `instance_path` 参数没有赋值，会适用下面默认的位置:

-   已卸载的模块::

        /myapp.py
        /instance

-   已卸载的包::

        /myapp
            /__init__.py
        /instance

-   安装过的模块或者包::

        $PREFIX/lib/python2.X/site-packages/myapp
        $PREFIX/var/myapp-instance

    ``$PREFIX`` 是你 Python 安装的前缀。这个前缀可以是 ``/usr`` 或者你 virtualenv 的路径。
    你可以打印 ``sys.prefix`` 的值来查看前缀被设置成了什么。


既然配置对象提供从相对文件名来载入配置的方式，那么我们也使得它从相对实例 路径的文件名加载成为可能，
如果你想这样做。配置文件中的相对路径的行为可以 在“相对应用的根目录”（默认）和 “相对实例文件夹”中切换，
后者通过应用构造函 数的 `instance_relative_config` 开关实现::

    app = Flask(__name__, instance_relative_config=True)

这里有一个配置 Flask 来从模块预载入配置并覆盖配置文件夹中配置文件(如果存在)的完整例子::

    app = Flask(__name__, instance_relative_config=True)
    app.config.from_object('yourapplication.default_settings')
    app.config.from_pyfile('application.cfg', silent=True)

实例文件夹的路径可以在 :attr:`Flask.instance_path` 找到。 Flask 也提供了 一个打开实例文件夹中文件的捷径，
就是 :meth:`Flask.open_instance_resource`  。

两者使用的示例::

    filename = os.path.join(app.instance_path, 'application.cfg')
    with open(filename) as f:
        config = f.read()

    # or via open_instance_resource:
    with app.open_instance_resource('application.cfg') as f:
        config = f.read()

