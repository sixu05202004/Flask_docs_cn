.. _tutorial-templates:

Step 6: 模版
=====================

现在我们应该开始编写模版。如果我们现在请求 URLs，我们将会得到一个 Flask 无法找到模版的异常。
模版使用 `Jinja2`_ 语言以及默认开启自动转义。这就意味着除非你使用 :class:`~flask.Markup` 标记或在模板中使用 ``|safe`` 过滤器，
否则 Jinja2 会确保特殊字符比如 ``<`` 或 ``>`` 被转义成等价的 XML 实体。

我们也使用模版继承使得在网站的所有页面中重用布局成为可能。

请把如下的模版放入 `templates` 文件夹：

.. _Jinja2: http://jinja.pocoo.org/2/documentation/templates

layout.html
-----------

这个模板包含 HTML 主体结构，标题和一个登录链接（或者当用户已登入则提供注销）。如果有闪现信息的话它也将显示
闪现信息。``{% block body %}`` 块能够被子模版中的同样名字( ``body`` )的块替代。

:class:`~flask.session` 字典在模版中同样可用的，你能用它检查用户是否登录。注意在 Jinja 中你可以访问不存在的对象/字典属性或成员，
如同下面的代码， 即便 ``'logged_in'`` 键不存在，仍然可以正常工作：

.. sourcecode:: html+jinja

    <!doctype html>
    <title>Flaskr</title>
    <link rel=stylesheet type=text/css href="{{ url_for('static', filename='style.css') }}">
    <div class=page>
      <h1>Flaskr</h1>
      <div class=metanav>
      {% if not session.logged_in %}
        <a href="{{ url_for('login') }}">log in</a>
      {% else %}
        <a href="{{ url_for('logout') }}">log out</a>
      {% endif %}
      </div>
      {% for message in get_flashed_messages() %}
        <div class=flash>{{ message }}</div>
      {% endfor %}
      {% block body %}{% endblock %}
    </div>

show_entries.html
-----------------

这个模版继承了上面的 `layout.html` 模版用来显示信息。注意 `for` 遍历了所有我们用 :func:`~flask.render_template` 
函数传入的信息。我们同样告诉表单提交到 `add_entry` 函数且使用 `HTTP` 的 `POST` 方法：

.. sourcecode:: html+jinja

    {% extends "layout.html" %}
    {% block body %}
      {% if session.logged_in %}
        <form action="{{ url_for('add_entry') }}" method=post class=add-entry>
          <dl>
            <dt>Title:
            <dd><input type=text size=30 name=title>
            <dt>Text:
            <dd><textarea name=text rows=5 cols=40></textarea>
            <dd><input type=submit value=Share>
          </dl>
        </form>
      {% endif %}
      <ul class=entries>
      {% for entry in entries %}
        <li><h2>{{ entry.title }}</h2>{{ entry.text|safe }}
      {% else %}
        <li><em>Unbelievable.  No entries here so far</em>
      {% endfor %}
      </ul>
    {% endblock %}

login.html
----------

最后是登录模板，基本上只显示一个允许用户登录的表单：

.. sourcecode:: html+jinja

    {% extends "layout.html" %}
    {% block body %}
      <h2>Login</h2>
      {% if error %}<p class=error><strong>Error:</strong> {{ error }}{% endif %}
      <form action="{{ url_for('login') }}" method=post>
        <dl>
          <dt>Username:
          <dd><input type=text name=username>
          <dt>Password:
          <dd><input type=password name=password>
          <dd><input type=submit value=Login>
        </dl>
      </form>
    {% endblock %}

继续浏览 :ref:`tutorial-css` 。
