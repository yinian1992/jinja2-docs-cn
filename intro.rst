介绍
============

这里是 Jinja2 通用模板语言的文档。 Jinja2 在其是一个 Python 2.4 库之前，被设计
为是灵活、快速和安全的。

如果你接触过其它的基于文本的模板语言，比如 Smarty 或 Django ，那么 Jinja2 会让你有
宾至如归的感觉。Jinja2 通过坚持 Python 原则来保证对设计者和开发者友好，为模板环
境添加有帮助的功能。

预备知识
-------------

Jinja2 需要至少 **Python 2.4** 版本来运行。此外，如果你使用 Python 2.4 ，一个可
以创建 python 扩展的可用的 C 编译器会为调试器安装。

如果你没有一个可用的 C 编译器，并且你视图安装带调试支持的源码版本，你会得到一个
编译器错误。
If you don't have a working C-compiler and you are trying to install the source

.. _ctypes: http://python.net/crew/theller/ctypes/


安装
------------

条条大路通 Jinja2 。如果你不确定怎么做，用 Python egg 或 tarball 吧。

作为一个 Python egg （通过 easy_install）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你可以用 `easy_install`_ 或 `pip`_ 安装最新的版本的 Jinja2::

    sudo easy_install Jinja2
    sudo pip install Jinja2

这会在你的 Python 安装中的 site-packages 目录安装一个 Jinja2 egg 。

（如果你在 Windows 的命令行中安装，省略 `sudo` 并且确保你用管理员权限运行
命令行）

从 tarball 版本安装
~~~~~~~~~~~~~~~~~~~~~~~~~

1.  从 `download page`_ 下载最新的 tarball
2.  解包 tarball
3.  ``sudo python setup.py install``

注意这需要你已经安装了 setuptools 或 `distribute`_ ，首选后者。

这会在你 Python 安装的 site-packages 目录安装 Jinja2 。

.. _distribute: http://pypi.python.org/pypi/distribute

安装开发版本
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  安装 `git`_
2.  ``git clone git://github.com/mitsuhiko/jinja2.git``
3.  ``cd jinja2``
4.  ``ln -s jinja2 /usr/lib/python2.X/site-packages``

作为第四步的替代选择，你也可以执行 ``python setup.py develop`` ，这会通过
disbribute 在开发模式下安装包。这样也有编译 C 扩展的优势。

.. _download page: http://pypi.python.org/pypi/Jinja2
.. _setuptools: http://peak.telecommunity.com/DevCenter/setuptools
.. _easy_install: http://peak.telecommunity.com/DevCenter/EasyInstall
.. _pip: http://pypi.python.org/pypi/pip
.. _git: http://git-scm.org/


加速 MarkupSafe
~~~~~~~~~~~~~~~~~~~~~~~~~~

从 2.5.1 开始， Jinja2 会检查是否安装 `MarkupSafe`_ 模块。如果它找到了，
它会用这个模块的 Markup 类来代替自带的。 `MarkupSafe` 替换 Jinja2 中附带的
老的加速模块，其优势在于更好的安装脚本，自动试图安装 C 的版本并在不可行时
漂亮地退化到纯 Python 实现的版本。

MarkupSafe 的 C 实现要快得多，并推荐用于 Jinja2 自动转义。

.. _MarkupSafe: http://pypi.python.org/pypi/MarkupSafe


启用调试支持模块
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

默认 Jinja2 不会编译调试支持模块。如果你没有 Python 头文件或可用的编译器，
启用它会失败。这当你在 Windows 上安装 Jinja2 是很常见的情况。

由于调试模式只对 Python 2.4 是必要的，所以你不需要这么做，除非你在运行
2.4::

    sudo python setup.py --with-debugsupport install


基本 API 使用
---------------

本节简要介绍 Jinja2 模板的 Python API 。

最基本的方式就是通过 :class:`~jinja2.Template` 创建一个模板并渲染它。
如果你的模板不是从字符串加载，而是文件系统或别的数据源，无论如何这都不
是推荐的方式:

>>> from jinja2 import Template
>>> template = Template('Hello {{ name }}!')
>>> template.render(name='John Doe')
u'Hello John Doe!'

通过创建一个 :class:`~jinja2.Template` 的实例，你会得到一个新的模板对象，提供一
个名为 :meth:`~jinja2.Template.render` 的方法，该方法在有字典或关键字参数时调用
扩充模板。字典或关键字参数会被传递到模板，即模板“上下文”。


如你所见， Jinja2 内部使用 unicode 并且返回值也是 unicode 字符串。所以确
保你的应用里也确实使用 unicode 。


实验性的 Python 3 支持
-----------------------------

Jinja 2.3 带来 Python 3 的实验性支持。这意味着在新版本上，所有的单元测试
都会通过，但是仍有一些小 bug 和不一致的行为。如果你发现任何 bug ，请向
`Jinja bug tracker`_ 提供反馈。

也请记住本文档是为 Python 2 编撰的，你会需要手动把示例代码转换为 Python 3
的语法。


.. _Jinja bug tracker: http://github.com/mitsuhiko/jinja2/issues
