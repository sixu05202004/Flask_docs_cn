.. _tutorial-views:

Step 5: 视图函数
==========================

现在数据库连接已经工作我们可以开始编写视图函数。我们需要四个视图函数：

显示条目
------------

这个视图显示所有存储在数据库中的条目。它监听者应用的根地址以及将会从数据库中查询标题和内容。
id值最大的条目（最新的条目）将在最前面。从游标返回的行是按 select 语句中声明的列组织的元组。
对于像我们这样的小应用是足够的，但是你可能要把它们转换成字典。如果你对如何转换成字典感兴趣的话，
请查阅 :ref:`easy-querying` 例子。

视图函数将会把条目作为字典传入 `show_entries.html` 模版以及返回渲染结果::

    @app.route('/')
    def show_entries():
        cur = g.db.execute('select title, text from entries order by id desc')
        entries = [dict(title=row[0], text=row[1]) for row in cur.fetchall()]
        return render_template('show_entries.html', entries=entries)

添加新条目
-------------

这个视图允许登录的用户添加新的条目。它只回应 `POST` 请求，实际的表单是显示在 `show_entries` 页面。
如果一些工作正常的话，我们用 :func:`~flask.flash` 向下一个请求闪现一条信息并且跳转回 `show_entries` 页::

    @app.route('/add', methods=['POST'])
    def add_entry():
        if not session.get('logged_in'):
            abort(401)
        g.db.execute('insert into entries (title, text) values (?, ?)',
                     [request.form['title'], request.form['text']])
        g.db.commit()
        flash('New entry was successfully posted')
        return redirect(url_for('show_entries'))

注意我们这里检查用户登录情况( `logged_in` 键存在会话中，并且为 `True` )。

.. admonition:: 安全提示

   确保像上面例子中一样，使用问号标记来构建 SQL 语句。否则，当你使用格式化字符串构建 SQL 语句时，
   你的应用容易遭受 SQL 注入。
   更多的信息请看 :ref:`sqlite3` 。

登录和注销
----------------

这些函数是用于用户登录以及注销。依据在配置中的值登录时检查用户名和密码并且在会话中设置 `logged_in` 键值。
如果用户成功登录，`logged_in` 键值被设置成 `True` ，并跳转回 `show_entries` 页。此外，会有消息闪现来提示用户登入成功。 如果发生一个错误，模板会通知，并提示重新登录::

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        error = None
        if request.method == 'POST':
            if request.form['username'] != app.config['USERNAME']:
                error = 'Invalid username'
            elif request.form['password'] != app.config['PASSWORD']:
                error = 'Invalid password'
            else:
                session['logged_in'] = True
                flash('You were logged in')
                return redirect(url_for('show_entries'))
        return render_template('login.html', error=error)

另一方面，注销函数从会话中移除了 `logged_in` 键值。这里我们使用一个大绝招：如果你使用字典的 :meth:`~dict.pop` 方法并传入第二个参数（默认）， 这个方法会从字典中删除这个键，如果这个键不存在则什么都不做。这很有用，因为 我们不需要检查用户是否已经登入。

::

    @app.route('/logout')
    def logout():
        session.pop('logged_in', None)
        flash('You were logged out')
        return redirect(url_for('show_entries'))

继续浏览 :ref:`tutorial-templates` 。
