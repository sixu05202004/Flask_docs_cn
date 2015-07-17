.. _installation:

安装
============

Flask 依赖两个外部库， `Werkzeug
<http://werkzeug.pocoo.org/>`_ 和 `Jinja2 <http://jinja.pocoo.org/2/>`_。Werkzeug
是一个 WSGI 工具集，它是 web 应用程序和用于开发和部署的服务器之间的标准接口。Jinja2 负责渲染模板。

因此怎样才能快速地安装这一切了？你有很多种方法去安装，但是最简单粗暴的方式就是 virtualenv，
让我们首先来看看它。

你首先需要 Python 2.6 或更高的版本，所以请确认有一个最新的 Python 2.x 安装。对于在 Python 3 中使用 Flask，请参看 :ref:`python3`。


.. _virtualenv:

virtualenv
----------

也许 Virtualenv 是你在开发中最愿意使用的，如果你在生产机器上有 shell 权限的时候，你也会愿意用上 virtualenv。

virtualenv 解决了什么问题？如果你像我一样喜欢 Python 的话，有很多机会在基于 Flask 的 web 应用外的其它项目上使用 Python。
然而项目越多，越有可能在不同版本的 python，或者至少在不同 python 库的版本上工作。
我们需要面对这样的事实：库破坏向后兼容性的情况相当常见，而且零依赖的正式应用也不大可能存在。
如此，当你的项目中的两个或更多出现依赖性冲突，你会怎么做？

Virtualenv 的出现解决这一切！Virtualenv 能够允许多个不同版本的 Python 安装，每一个服务于各自的项目。
它实际上并没有安装独立的 Python 副本，只是提供了一种方式使得环境保持独立。让我们见识下 virtualenv 怎么工作的。

如果你在 Mac OS X 或 Linux下，下面两条命令可能会适用::

    $ sudo easy_install virtualenv

或者更好的::

    $ sudo pip install virtualenv

上述的命令会在你的系统中安装 virtualenv。它甚至可能会出现在包管理器中。如果你使用 Ubuntu ，请尝试::

    $ sudo apt-get install python-virtualenv

如果是在 Windows 下并且没有安装 `easy_install` 命令，你首先必须安装 `easy_install` 。
要想获取更多的安装信息，请查看 :ref:`windows-easy-install` 。一旦安装好 `easy_install` ，
运行上述的命令，但是要去掉 sudo 前缀。

一旦成功安装 virtualenv，运行 shell 创建自己的环境。我通常会创建一个项目文件夹，其下创建 `venv` 文件夹::

    $ mkdir myproject
    $ cd myproject
    $ virtualenv venv
    New python executable in venv/bin/python
    Installing distribute............done.

现在，只要你想要在某个项目上工作，只要激活相应的环境。在 OS X 和 Linux 下，按如下做::

    $ . venv/bin/activate

如果你是个 Windows 用户，下面的命令行是为你准备的::

    $ venv\scripts\activate

无论哪种方式，你现在能够使用你的 virtualenv (注意你的 shell 提示符显示的是活动的环境)。

现在你只需要键入以下的命令来激活你的 virtualenv 中的 Flask::

    $ pip install Flask

几秒后，一切就为你准备就绪。


全局安装
------------------------

这样也是可能的，尽管我不推荐。只需要以 root 权限运行 pip::

    $ sudo pip install Flask

(在 Windows 系统上，在管理员权限的命令提示符中运行这条命令，不需要 sudo。)


体验最新的 Flask (Living on the Edge)
--------------------------------------

如果你想要用最新版的 Flask 干活，这里有两种方式：你可以使用 pip 拉取开发版本，
或让它操作一个 git checkout。无论哪种方式，依然推荐使用 virtualenv。

在一个新的 virtualenv 上获取一个 git checkout，在开发模式下运行::

    $ git clone http://github.com/mitsuhiko/flask.git
    Initialized empty Git repository in ~/dev/flask/.git/
    $ cd flask
    $ virtualenv venv --distribute
    New python executable in venv/bin/python
    Installing distribute............done.
    $ . venv/bin/activate
    $ python setup.py develop
    ...
    Finished processing dependencies for Flask

这会拉取依赖关系并激活 git head 作为 virtualenv 中的当前版本。然后你只需要执行 ``git pull
origin`` 来升级到最新版本。

没有 git 下获取最新的开发版本，需要这样做::

    $ mkdir flask
    $ cd flask
    $ virtualenv venv --distribute
    $ . venv/bin/activate
    New python executable in venv/bin/python
    Installing distribute............done.
    $ pip install Flask==dev
    ...
    Finished processing dependencies for Flask==dev

.. _windows-easy-install:

Windows 下的 `pip` 和 `distribute`
-----------------------------------

在 Windows 系统下，安装 `easy_install` 有些棘手，但是仍然很简单。最简单的方式是下载 
`distribute_setup.py`_ 文件接着运行它。运行这个文件最简单的方式就是打开下载文件夹接着双击这个文件。

接着，把 Python 的 Scripts 文件夹添加到 `PATH` 环境变量来，这样 `easy_install` 命令和其它 Python 脚本就加入到了命令行自动搜索的路径。做法是：右键单击桌面上或是“开始”菜单中的“我的电脑”图标，选择“属性”，
然后单击“高级系统设置”（在 Windows XP 中，单击“高级”选项卡），然后单击“环境变量”按钮， 
最后双击“系统变量”栏中的“Path”变量，并加入你的 Python 解释器的 Scripts 文件夹。
确保你用分号把它和现有的值分隔开。假设你使用 Python 2.7 且为默认目录，添加下面的值::


    ;C:\Python27\Scripts

这样就完成了！为了检测是否正常工作，打开命令提示符执行 ``easy_install``。在 Windows Vista
或者 Windows 7 下如果开启了用户账户控制，它应该提示需要管理员权限。

现在已经安装好 ``easy_install``，你能使用它来安装 ``pip``::

    > easy_install pip


.. _distribute_setup.py: http://python-distribute.org/distribute_setup.py
