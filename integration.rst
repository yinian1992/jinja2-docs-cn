集成
===========

Jinja2 提供了一些代码来继承到其它工具，诸如框架、 `Babel`_ 库或你偏好的编辑器
的奇特的代码高亮。这里是包含的这些的简要介绍。

帮助继承的文件在 `这里 <https://github.com/mitsuhiko/jinja2/master/ext>`_ 可
用。

.. _babel-integration:

Babel 集成
-----------------

Jinja 提供了用 `Babel`_ 抽取器从模板中抽取 gettext 消息的支持，抽取器的接入点
名为 `jinja2.ext.babel_extract` 。 Babel 支持的被作为 :ref:`i18n-extension` 的
一部分实现。

Gettext 消息从 `trans` 标签和代码表达式中抽取。

要从模板中抽取 gettext 消息，项目需要在它的 Babel 抽取方法 `mapping file`_ 中
有一个 Jinja2 节:

.. sourcecode:: ini

    [jinja2: **/templates/**.html]
    encoding = utf-8

:class:`Environment` 的语法相关选项也可作为 mapping file 的配置值。例如告知
抽取器模板使用 ``%`` 作为 `line_statement_prefix` 你可以这样写:

.. sourcecode:: ini

    [jinja2: **/templates/**.html]
    encoding = utf-8
    line_statement_prefix = %

:ref:`jinja-extensions` 可能也被定义为传递一个逗号分割的导入路径列表作为
`extensions` 值。 i18n 扩展会被自动添加。

.. versionchanged:: 2.7

   直到 2.7 模板语法错误始终被忽略。因为许多人在模板文件夹中放置非模板的
   html 文件，而这会随机报错，所以如此设定。假定是无论如何测试套件会捕获
   模板中的语法错误。如果你不想要这个行为，你可以在设置中添加
   ``slient=flase`` ，异常会被传播。

.. _mapping file: http://babel.edgewall.org/wiki/Documentation/messages.html#extraction-method-mapping-and-configuration

Pylons
------

从 `Pylons`_ 0.9.7 开始，集成 Jinja 到 Pylons 驱动的应用令人难以置信的简单。

模板引擎在 `config/environment.py` 中配置。为 Jinja2 的配置看起来是这样::

    from jinja2 import Environment, PackageLoader
    config['pylons.app_globals'].jinja_env = Environment(
        loader=PackageLoader('yourapplication', 'templates')
    )

之后，你可以用 `pylons.templating` 模块中的 `render_jinja` 函数渲染 Jinja 模板。

此外，设置 Pylons 的 `c` 对象为严格模式是个好主意。按照默认，访问任何 `c` 对象
上不存在的属性会返回一个空字符串而不是一个未定义对象。更改这个，只需要使用这个
片段并添加到你的 `config/environment.py` 中::

    config['pylons.strict_c'] = True

.. _Pylons: http://www.pylonshq.com/

TextMate
--------

在 Jinja2 项目根目录的 `ext` 文件夹中，有一个 TextMate 的 bundle 来提供 Jinja1
和 Jinja2 的基于文本的模板的语法高亮，同样也支持 HTML 。它也包含了一些常用的片
段。

Vim
---

同样，在 Jinja2 项目的根目录下的 `ext` 文件夹中的 Vim-scripts 目录有一个 `Vim`_
的语法插件。 `这个脚本 <http://www.vim.org/scripts/script.php?script_id=1856>`_
支持 Jinja1 和 Jinja2 。安装后， `jinja` 和 `htmljinja` 两种文件类型可用。前者
给基于文本的模板，后者给 HTML 模板。

把这些文件复制到你的 `syntax` 文件夹。

.. _Babel: http://babel.edgewall.org/
.. _Vim: http://www.vim.org/
