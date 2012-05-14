.. _jinja-extensions:

扩展
==========

Jinja2 支持扩展来添加过滤器、测试、全局变量或者甚至是处理器。扩展的主要动力是
把诸如添加国际化支持的常用代码迁移到一个可重用的类。


添加扩展
-----------------

扩展在 Jinja2 环境创建时被添加。一旦环境被创建，就不能添加额外的扩展。要添加
一个扩展，传递一个扩展类或导入路径的列表到 :class:`Environment` 构造函数的
`environment` 参数。下面的例子创建了一个加载了 i18n 扩展的 Jinja2 环境::

    jinja_env = Environment(extensions=['jinja2.ext.i18n'])


.. _i18n-extension:

i18n 扩展
--------------

**Import name:** `jinja2.ext.i18n`

Jinja2 当前只附带一个扩展，就是 i18n 扩展。它可以与 `gettext`_ 或 `babel`_
联合使用。如果启用了 i18n 扩展， Jinja2 提供了 `trans` 语句来标记被其包裹的
字符串为可翻译的，并调用 `gettext` 。

在启用虚拟的 `_` 函数后，之后的 `gettext` 调用会被添加到环境的全局变量。那么
一个国际化的应用应该不仅在全局，以及在每次渲染中在命名空间中提供至少一个
`gettext`  或可选的 `ngettext` 函数。

环境方法
~~~~~~~~~~~~~~~~~~~

在启用这个扩展后，环境提供下面的额外方法:

.. method:: jinja2.Environment.install_gettext_translations(translations, newstyle=False)

    在该环境中全局安装翻译。提供的翻译对象要至少实现 `uggettext` 和
    `ungettext` 。 `gettext.NullTranslations` 和 `gettext.GNUTranslations`
    类和 `Babel`_'s 的 `Translations` 类也被支持。

    .. versionchanged:: 2.5 添加了新样式的 gettext

.. method:: jinja2.Environment.install_null_translations(newstyle=False)

    安装虚拟的 gettext 函数。这在你想使应用为国际化做准备但还不想实现完整的
    国际化系统时很有用。

    .. versionchanged:: 2.5 添加了新样式的 gettext

.. method:: jinja2.Environment.install_gettext_callables(gettext, ngettext, newstyle=False)

    在环境中把给出的 `gettext` 和 `ngettext` 可调用量安装为全局变量。它们
    应该表现得几乎与标准库中的 :func:`gettext.ugettext` 和
    :func:`gettext.ungettext` 函数相同。

    如果激活了 `新样式` ，可调用量被包装为新样式的可调用量一样工作。更多
    信息见 :ref:`newstyle-gettext` 。

    .. versionadded:: 2.5

.. method:: jinja2.Environment.uninstall_gettext_translations()

    再次卸载翻译。

.. method:: jinja2.Environment.extract_translations(source)

    从给定的模板或源中提取本地化字符串。

    对找到的每一个字符串，这个函数生产一个 ``(lineno, function, message
    )`` 元组，在这里:

    * `lineno` 是这个字符串所在行的行号。
    * `function` 是 `gettext` 函数使用的名称（如果字符串是从内嵌的 Python
      代码中抽取的）。
    * `message` 是字符串本身（一个 `unicode` 对象，在函数有多个字符串参数
      时是一个 `unicode` 对象的元组）。

    如果安装了 `Babel`_ ， :ref:`Babel 集成 <babel-integration>` 可以用来为
    babel 抽取字符串。

对于一个对多种语言可用而对所有用户给出同一种的语言的 web 应用（例如一个法国社
区安全了一个多种语言的论坛软件）可能会一次性加载翻译并且在环境生成时把翻译方
法添加到环境上::

    translations = get_gettext_translations()
    env = Environment(extensions=['jinja2.ext.i18n'])
    env.install_gettext_translations(translations)

`get_get_translations` 函数会返回当前配置的翻译器。（比如使用 `gettext.find`
）

模板设计者的 `i18n` 扩展使用在 :ref:`模板文档 <i18n-in-templates>` 中有描述。

.. _gettext: http://docs.python.org/dev/library/gettext
.. _Babel: http://babel.edgewall.org/

.. _newstyle-gettext:

新样式 Gettext
~~~~~~~~~~~~~~~~

.. versionadded:: 2.5

从版本 2.5 开始你可以使用新样式的 gettext 调用。这些的启发源于 trac 的内部
gettext 函数并且完全被 babel 抽取工具支持。如果你不使用 Babel 的抽取工具，
它可能不会像其它抽取工具预期的那样工作。

标准 gettext 调用和新样式的 gettext 调用有什么区别？通常，它们要输入的东西
更少，出错率更低。并且如果在自动转义环境中使用它们，它们也能更好地支持自动
转义。这里是一些新老样式调用的差异:

标准 gettext:

.. sourcecode:: html+jinja

    {{ gettext('Hello World!') }}
    {{ gettext('Hello %(name)s!')|format(name='World') }}
    {{ ngettext('%(num)d apple', '%(num)d apples', apples|count)|format(
        num=apples|count
    )}}

新样式看起来是这样:    

.. sourcecode:: html+jinja

    {{ gettext('Hello World!') }}
    {{ gettext('Hello %(name)s!', name='World') }}
    {{ ngettext('%(num)d apple', '%(num)d apples', apples|count) }}

新样式 gettext 的优势是你需要输入的更少，并且命名占位符是强制的。后者看起
来似乎是缺陷，但解决了当翻译者不能切换两个占位符的位置时经常勉励的一大堆
麻烦。使用新样式的 gettext ，所有的格式化字符串看起来都一样。

除此之外，在新样式 gettext 中，如果没有使用占位符，字符串格式化也会被使用，
这使得所有的字符串表现一致。最后，不仅是新样式的 gettext 调用可以妥善地为
解决了许多转义相关问题的自动转义标记字符串，许多模板也在使用自动转义时体验
了多次。

表达式语句
--------------------

**Import name:** `jinja2.ext.do`

“do”又叫做表达式语句扩展，向模板引擎添加了一个简单的 `do` 标签，其工作如同
一个变量表达式，只是忽略返回值。

.. _loopcontrols-extension:

循环控制
-------------

**Import name:** `jinja2.ext.loopcontrols`

这个扩展添加了循环中的 `break` 和 `continue` 支持。在启用它之后， Jinja2
提供的这两个关键字如同 Python 中那样工作。

.. _with-extension:

With 语句
--------------

**Import name:** `jinja2.ext.with_`

.. versionadded:: 2.3

这个扩展添加了 with 关键字支持。使用这个关键字可以在模板中强制一块嵌套的
作用域。变量可以在 with 语句的块头中直接声明，或直接在里面使用标准的 `set`
语句。

.. _autoescape-extension:

自动转义扩展
--------------------

**Import name:** `jinja2.ext.autoescape`

.. versionadded:: 2.4

自动转义扩展允许你在模板内开关自动转义特性。如果环境的
:attr:`~Environment.autoescape` 设定为 `False` ，它可以被激活。如果是 `True`
可以被关闭。这个设定的覆盖是有作用域的。

.. _writing-extensions:

编写扩展
------------------

.. module:: jinja2.ext

你可以编写扩展来向 Jinja2 中添加自定义标签。这是一个不平凡的任务，而且通常不需
要，因为默认的标签和表达式涵盖了所有常用情况。如 i18n 扩展是一个扩展有用的好例
子，而另一个会是碎片缓存。

当你编写扩展时，你需要记住你在与 Jinja2 模板编译器一同工作，而它并不验证你传递
到它的节点树。如果 AST 是畸形的，你会得到各种各样的编译器或运行时错误，这调试起
来极其可怕。始终确保你在使用创建正确的节点。下面的 API 文档展示了有什么节点和如
何使用它们。

示例扩展
~~~~~~~~~~~~~~~~~

下面的例子用 `Werkzeug`_ 的缓存 contrib 模块为 Jinja2 实现了一个 `cache` 标签:

.. literalinclude:: cache_extension.py
    :language: python

而这是你在环境中使用它的方式::

    from jinja2 import Environment
    from werkzeug.contrib.cache import SimpleCache

    env = Environment(extensions=[FragmentCacheExtension])
    env.fragment_cache = SimpleCache()

之后，在模板中可以标记块为可缓存的。下面的例子缓存一个边栏 300 秒:

.. sourcecode:: html+jinja

    {% cache 'sidebar', 300 %}
    <div class="sidebar">
        ...
    </div>
    {% endcache %}

.. _Werkzeug: http://werkzeug.pocoo.org/

扩展 API
~~~~~~~~~~~~~

扩展总是继承 :class:`jinja2.ext.Extension` 类:

.. autoclass:: Extension
    :members: preprocess, filter_stream, parse, attr, call_method

    .. attribute:: identifier

        扩展的标识符。这始终是扩展类的真实导入名，不能被修改。

    .. attribute:: tags

        如果扩展实现自定义标签，这是扩展监听的标签名的集合。

解析器 API
~~~~~~~~~~~~~~

传递到 :meth:`Extension.parse` 的解析器提供解析不同类型表达式的方式。下
面的方法可能会在扩展中使用:

.. autoclass:: jinja2.parser.Parser
    :members: parse_expression, parse_tuple, parse_assign_target,
              parse_statements, free_identifier, fail

    .. attribute:: filename

        解析器处理的模板文件名。这 **不是** 模板的加载名。加载名见
        :attr:`name` 。对于不是从文件系统中加载的模板，这个值为 `None` 。

    .. attribute:: name

        模板的加载名。

    .. attribute:: stream

        当前的 :class:`~jinja2.lexer.TokenStream` 。

.. autoclass:: jinja2.lexer.TokenStream
   :members: push, look, eos, skip, next, next_if, skip_if, expect

   .. attribute:: current

        当前的 :class:`~jinja2.lexer.Token` 。

.. autoclass:: jinja2.lexer.Token
    :members: test, test_any

    .. attribute:: lineno

        token 的行号。

    .. attribute:: type

        token 的类型。这个值是被禁锢的，所以你可以用 `is` 运算符同任意字符
        串比较。

    .. attribute:: value

        token 的值。

同样，在词法分析模块中也有一个实用函数可以计算字符串中的换行符数目::

.. autofunction:: jinja2.lexer.count_newlines

AST
~~~

AST（抽象语法树: Abstract Syntax Tree）用于表示解析后的模板。它有编译器之后
转换到可执行的 Python 代码对象的节点构建。提供自定义语句的扩展可以返回执行自
定义 Python 代码的节点。

下面的清单展示了所有当前可用的节点。 AST 在 Jinja2 的各个版本中有差异，但会向
后兼容。

更多信息请见 :meth:`jinja2.Environment.parse` 。

.. module:: jinja2.nodes

.. jinjanodes::

.. autoexception:: Impossible
