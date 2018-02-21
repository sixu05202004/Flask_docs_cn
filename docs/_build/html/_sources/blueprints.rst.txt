.. _blueprints:

用蓝图实现模块化应用
====================================

.. versionadded:: 0.7

Flask 使用了 *蓝图* 的概念在一个应用或者跨应用中构建应用组件以及支持通用模式。 蓝图很好地简化了大型应用工作的方式，并提供给 Flask 扩展在应用上注册操作的核心方法。 一个 :class:`Blueprint` 对象与 :class:`Flask` 应用对象的工作方式很像，但它确实不是一个应用， 而是一个描述如何构建或扩展应用的 *蓝图* 。

为什么用蓝图？
---------------

Flask中的蓝图旨在针对这些情况:

* 把一个应用分解成一系列的蓝图。对于大型的应用是理想化的；一个项目能实例化一个应用， 初始化一些扩展，以及注册一系列的蓝图。
* 以一个 URL 前缀和/或子域在一个应用上注册蓝图。 URL 前缀/子域名中的参数即成为这个蓝图下的所有视图函数的共同的视图参数（默认情况下）。
* 在一个应用中用不同的 URL 规则多次注册一个蓝图。
* 通过蓝图提供模板过滤器、静态文件、模板和其它功能。一个蓝图不一定要实现应用或视图函数。
* 初始化一个 Flask 扩展时，在这些情况中注册蓝图。

Flask 中的蓝图不是即插应用，因为它实际上并不是一个应用 -- 它是可以注册，甚至可以多次注册到应用上的操作集合。为什么不使用多个应用对象？你可以做到那样 （见 :ref:`app-dispatch` ），但是你的应用会有分开的配置，并在 WSGI 层管理。

蓝图作为 Flask 层提供分割的替代，共享应用配置，并且可以更改所注册的应用对象。其短板是你不能在应用创建后撤销注册一个蓝图而不销毁整个应用对象。

蓝图的概念
-------------------------

蓝图的基本设想是它们记录注册到一个应用时的操作执行情况。 当从一个端点到另一端分发请求和生成 URL 时，Flask 关联视图函数和蓝图。

第一个蓝图
------------------

这看起来像是一个非常基本的蓝图。在这个案例中，我们想要实现一个简单渲染静态模板的蓝图::

    from flask import Blueprint, render_template, abort
    from jinja2 import TemplateNotFound

    simple_page = Blueprint('simple_page', __name__,
                            template_folder='templates')

    @simple_page.route('/', defaults={'page': 'index'})
    @simple_page.route('/<page>')
    def show(page):
        try:
            return render_template('pages/%s.html' % page)
        except TemplateNotFound:
            abort(404)

当我们使用 ``\@simple_page.route`` 装饰器绑定函数时，蓝图会记录下所登记的 `show` 函数。当以后在应用中注册蓝图时，这个函数会被注册到应用中。此外，它会给函数名加上由 :class:`Blueprint` 的构造函数中给出的蓝图的名称作为前缀 (在此例中是 ``simple_page`` )。

注册蓝图
----------------------

如何注册蓝图了？像这样::

    from flask import Flask
    from yourapplication.simple_page import simple_page

    app = Flask(__name__)
    app.register_blueprint(simple_page)

如果你检查注册到应用的规则，你将会发现这些::

    [<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
     <Rule '/<page>' (HEAD, OPTIONS, GET) -> simple_page.show>,
     <Rule '/' (HEAD, OPTIONS, GET) -> simple_page.show>]

第一个显然是来自应用自身，用于静态文件。其它的两个用于 ``simple_page`` 蓝图中的 `show` 函数。如你所见，它们的前缀是蓝图的名称，并且用一个点(``.``)来分割。

不过，蓝图也可以在不同的位置挂载::

    app.register_blueprint(simple_page, url_prefix='/pages')

果然，生成这些规则::

    [<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
     <Rule '/pages/<page>' (HEAD, OPTIONS, GET) -> simple_page.show>,
     <Rule '/pages/' (HEAD, OPTIONS, GET) -> simple_page.show>]

总之，你可以多次注册蓝图，但是不一定每个蓝图都能正确响应。 是否能够多次注册实际上取决于你的蓝图是如何编写的，是否能根据不同的位置做出正确的响应。

蓝图资源
-------------------

蓝图也可以提供资源。有时候你会只为它提供的资源而引入一个蓝图。

蓝图资源文件夹
`````````````````````````

与常规应用一样，蓝图被认为是包含在一个文件夹中。虽然多个蓝图可以源自相同的文件夹中， 它并不必须是这种情况并且通常不建议这样做。

这个文件夹会从 :class:`Blueprint` 的第二个参数中推断出来，通常是 `__name__` 。 这个参数决定对应蓝图的是哪个逻辑的 Python 模块或包。如果它指向一个存在的 Python 包，这个包（通常是文件系统中的文件夹）就是资源文件夹。如果是一个模块， 模块所在的包就是资源文件夹。你可以访问 :attr:`Blueprint.root_path` 属性来查看资源文件夹什么::

    >>> simple_page.root_path
    '/Users/username/TestProject/yourapplication'

你可以使用 :meth:`~Blueprint.open_resource` 函数快速性这个文件夹中打开资源::

    with simple_page.open_resource('static/style.css') as f:
        code = f.read()

静态文件
````````````

一个蓝图可以通过 `static_folder` 关键字参数提供一个指向文件系统上文件夹的路 径，来公开一个带有静态文件的文件夹。这可以是一个绝对路径，也可以是相对于蓝图文件夹的路径::

    admin = Blueprint('admin', __name__, static_folder='static')

默认情况下，路径最右边的部分就是它在 web 上所公开的地址。因为这里这个文件夹叫做 ``static`` ， 它会在蓝图 + ``/static`` 的位置上可用。也就是说，蓝图为 ``/admin`` 把静态文件夹注册到 ``/admin/static`` 。

最后是命名的 `blueprint_name.static` ，这样你可以生成它的 URL ，就像你对应用的静态文件夹所做的那样::

    url_for('admin.static', filename='style.css')

模板
`````````

如果你想要蓝图公开模板，你可以提供 :class:`Blueprint` 构造函数中的 `template_folder` 参数来实现::

    admin = Blueprint('admin', __name__, template_folder='templates')

像对待静态文件一样，路径可以是绝对的或是相对蓝图资源文件夹的。模板文件夹会 被加入到模板的搜索路径中，但是比实际的应用模板文件夹优先级低。 这样，你可以容易地在实际的应用中覆盖蓝图提供的模板。

那么当你有一个 ``yourapplication/admin`` 文件夹中的蓝图并且你想要渲染 ``'admin/index.html'`` 模板， 且你已经提供了 ``templates`` 作为 `template_folder` ，你需要这样创建文件: ``yourapplication/admin/templates/admin/index.html``。

构建 URLs
-------------

当你想要从一个页面链接到另一个页面，你可以像通常一个样使用 :func:`url_for` 函数，只是你要在 URL 的末端加上蓝图的名称和一个点(``.``)作为前缀::

    url_for('admin.index')

此外，如果你在一个蓝图的视图函数或是模板中想要从链接到同一蓝图下另一个端点， 你可以通过对端点的只加上一个点作为前缀来使用相对的重定向::

    url_for('.index')

这个案例中，它实际上链接到 ``admin.index`` ，假如请求被分派到任何其它的 admin 蓝图端点。
