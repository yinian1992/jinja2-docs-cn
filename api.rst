API
===

.. module:: jinja2
    :synopsis: public Jinja2 API

本文档描述 Jinja2 的 API 而不是模板语言。这对实现模板接口，而非创建 Jinja2
模板，是最有用的参考，


基础
------

Jinja2 使用一个名为 :class:`Environment` 的中心对象。这个类的实例用于存储配
置、全局对象，并用于从文件系统或其它位置加载模板。即使你通过:class:`Template`
类的构造函数用字符串创建模板，也会为你自动创建一个环境，尽管是共享的。

大多数应用在应用初始化时创建一个 :class:`Environment` 对象，并用它加载模板。
在某些情况下，如果使用多份配置，使用并列的多个环境无论如何是有用的。

配置 Jinja2 为你的应用加载文档的最简单方式看起来大概是这样::

    from jinja2 import Environment, PackageLoader
    env = Environment(loader=PackageLoader('yourapplication', 'templates'))

这会创建一个默认设定下的模板环境和一个在 `yourapplication` python 包中的
`templates` 文件夹中寻找模板的加载器。多个加载器是可用的，如果你需要从
数据库或其它资源加载模板，你也可以自己写一个。

你只需要调用 :meth:`get_template` 方法从这个环境中加载模板，并会返回已加载的
:class:`Template`::

    template = env.get_template('mytemplate.html')

用若干变量来渲染它，调用 :meth:`render` 方法::

    print template.render(the='variables', go='here')

使用一个模板加载器，而不是向 :class:`Template` 或
:meth:`Environment.from_string` 传递字符串，有许多好处。除了使用上便利，
也使得模板继承成为可能。


Unicode
-------

Jinja2 内部使用 Unicode ，这意味着你需要向渲染函数传递 Unicode 对象或只包含
ASCII 字符的字符串。此外，换行符按照默认 UNIX 风格规定行序列结束（ ``\n`` ）。

Python 2.x 支持两种表示字符串对象的方法。一种是 `str` 类型，另一种是
`unicode` 类型，它们都继承于 `basestring` 类型。不幸的是，默认的 `str` 不
应该用于存储基于文本的信息，除非只用到 ASCII 字符。在 Python 2.6 中，可以
在模块层指定 `unicode` 为默认值，而在 Python 3 中会是默认值。

要显式使用一个 Unicode 字符串，你需要给字符串字面量加上 `u` 前缀：
``u'Hänsel und Gretel sagen Hallo'`` 。这样 Python 会用当前模块的字符编码来
解码字符串，来把字符串存储为 Unicode 。如果没有指定编码，默认是 `ASCII` ，
这意味着你不能使用任何非 ASCII 的标识符。

在使用 Unicode 字面量的 Python 模块的首行或第二行添加下面的注释，来妥善设
置模块编码::

    # -*- coding: utf-8 -*-

我们推荐为 Python 模块和模板使用 utf-8 编码，因为在 utf-8 中，可以表示
Unicode 中的每个字符，并且向后兼容 ASCII 。对于 Jinja2 ，模板的默认编码
假定为 utf-8 。

用 Jinja2 来处理非 Unicode 数据是不可能的。这是因为 Jinja2 已经在语言层
使用了 Unicode 。例如 Jinja2 在表达式中把不间断空格视为有效的空格，这需要
获悉编码或操作一个 Unicode 字符串。

关于 Python 中 Unicode 的更多细节，请阅读完善的
`Unicode documentation`_ 。

另一件重要的事情是 Jinja2 如何处理模板中的字符串字面量。原生实现会对所有
字符串字面量使用 Unicode ，但在过去这是有问题的，因为一些库显式地检查它
们的类型是否为 `str` 。例如 `datetime.strftime` 不接受 Unicode 参数。
为了不彻底破坏它， Jinja2 对只有 ASCII 的字符串返回 `str`，而对其它返回
`unicode`:

>>> m = Template(u"{% set a, b = 'foo', 'föö' %}").module
>>> m.a
'foo'
>>> m.b
u'f\xf6\xf6'


.. _Unicode documentation: http://docs.python.org/dev/howto/unicode.html

高层 API
--------------

高层 API 即是你会在应用中用于加载并渲染模板的 API 。 :ref:`low-level-api`
相反，只在你想深入挖掘 Jinja2 或 :ref:`开发扩展 <jinja-extensions>` 时有用。


.. autoclass:: Environment([options])
    :members: from_string, get_template, select_template,
              get_or_select_template, join_path, extend, compile_expression,
              compile_templates, list_templates, add_extension

    .. attribute:: shared

        如果模板通过  :class:`Template` 构造函数创建，会自动创建一个环境。这
        些环境被创建为共享的环境，这意味着多个模板拥有相同的匿名环境。对所有
        模板共享环境，这个属性为 `True` ，反之为 `False` 。

    .. attribute:: sandboxed

        如果环境在沙箱中，这个属性为 `True` 。沙箱模式见文档中的
        :class:`~jinja2.sandbox.SandboxedEnvironment` 。

    .. attribute:: filters

        该环境的过滤器字典。只要没有加载过模板，添加新过滤器或删除旧的都是
        安全的。自定义过滤器见 :ref:`writing-filters` 。有效的过滤器名称见
        :ref:`identifier-naming` 。

    .. attribute:: tests

        该环境的测试函数字典。只要没有加载过模板，修改这个字典都是安全的。
        自定义测试见 see :ref:`writing-tests` 。有效的测试名见
        :ref:`identifier-naming` 。

    .. attribute:: globals

        一个全局变量字典。这些变量在模板中总是可用。只要没有加载过模板，修
        改这个字典都是安全的。更多细节见 :ref:`global-namespace` 。有效的
        对象名见 :ref:`identifier-naming` 。

    .. automethod:: overlay([options])

    .. method:: undefined([hint, obj, name, exc])

        为 `name` 创建一个新 :class:`Undefined` 对象。这对可能为某些操作返回
        未定义对象过滤器和函数有用。除了 `hint` ，为了良好的可读性，所有参数
        应该作为关键字参数传入。如果提供了 `hint` ，它被用作异常的错误消息，
        否则错误信息会由 `obj` 和 `name` 自动生成。 `exc` 为生成未定义对象而
        不允许未定义的对象时抛出的异常。默认的异常是 :exc:`UndefinedError` 。
        如果提供了 `hint` ， `name` 会被发送。

        创建一个未定义对象的最常用方法是只提供名称::

            return environment.undefined(name='some_name')

        这意味着名称 `some_name` 未被定义。如果名称来自一个对象的属性，把
        持有它的对象告知未定义对象对丰富错误消息很有意义::

            if not hasattr(obj, 'attr'):
                return environment.undefined(obj=obj, name='attr')

        更复杂的例子中，你可以提供一个 hint 。例如 :func:`first` 过滤器
        用这种方法创建一个未定义对象::

            return environment.undefined('no first item, sequence was empty')            

        如果 `name` 或 `obj` 是已知的（比如访问了了一个属性），它应该传递给
        未定义对象，即使提供了自定义的 `hint` 。这让未定义对象有可能增强错误
        消息。

.. autoclass:: Template
    :members: module, make_module

    .. attribute:: globals

        该模板的全局变量字典。修改这个字典是不安全的，因为它可能与其它模板或
        加载这个模板的环境共享。

    .. attribute:: name

        模板的加载名。如果模板从字符串加载，这个值为 `None` 。

    .. attribute:: filename

        模板在文件系统上的文件名，如果没有从文件系统加载，这个值为 `None` 。

    .. automethod:: render([context])

    .. automethod:: generate([context])

    .. automethod:: stream([context])


.. autoclass:: jinja2.environment.TemplateStream()
    :members: disable_buffering, enable_buffering, dump


自动转义
------------

.. versionadded:: 2.4

从 Jinja 2.4 开始，自动转义的首选途径就是启用 :ref:`autoescape-extension`
并为自动转义配置一个合适的默认值。这使得在单个模板基础上开关自动转义成为
可能（比如 HTML 对 文本）

这里推荐为以 ``.html`` 、 ``.htm`` 、 ``.xml`` 以及 ``.xhtml`` 的模板开启
自动转义 ，并对所有其它扩展名禁用::

    def guess_autoescape(template_name):
        if template_name is None or '.' not in template_name:
            return False
        ext = template_name.rsplit('.', 1)[1]
        return ext in ('html', 'htm', 'xml')

    env = Environment(autoescape=guess_autoescape,
                      loader=PackageLoader('mypackage'),
                      extensions=['jinja2.ext.autoescape'])

假设实现一个自动转义函数，确保你也视 `None` 为有效模板名接受。这会在从字符
串生成模板时传递。

可以用 `autoescape` 块在模板内临时地更改这种行为。（见
:ref:`autoescape-overrides` ）。


.. _identifier-naming:

标识符的说明
--------------------

Jinja2 使用正规的 Python 2.x 命名规则。有效的标识符必须匹配
``[a-zA-Z_][a-zA-Z0-9_]*`` 。事实上，当前不允许非 ASCII 字符。这个限制可能
会在 Python 3 充分规定 unicode 标识符后消失。

过滤器和测试会在独立的命名空间中查找，与标识符语法有细微区别。过滤器和测
试可以包含点，用于按主题给过滤器和测试分组。例如，把一个名为 `to.unicode`
的函数添加到过滤器字典是完全有效的。过滤器和测试标识符的正则表达式是
``[a-zA-Z_][a-zA-Z0-9_]*(\.[a-zA-Z_][a-zA-Z0-9_]*)*`` 。


未定义类型
---------------

这些类可以用作未定义类型。 :class:`Environment` 的构造函数接受一个可以是
那些类或一个 :class:`Undefined` 的自定义子类的 `undefined` 参数。无论何时，
这些对象创建或返回时，模板引擎都不能查出其名称或访问其属性。未定义值上的
某些操作之后是允许的，而其它的会失败。

最接近常规 Python 行为的是 `StrictUndefined` ，如果它是一个未定义对象，
它不允许除了测试之外的一切操作。

.. autoclass:: jinja2.Undefined()

    .. attribute:: _undefined_hint

        `None` 或给未定义对象的错误消息 unicode 字符串。

    .. attribute:: _undefined_obj

        `None` 或引起未定义对象创建的对象（例如一个属性不存在）。

    .. attribute:: _undefined_name

        未定义变量/属性的名称，如果没有此类信息，留为 `None` 。

    .. attribute:: _undefined_exception

        未定义对象想要抛出的异常。这通常是 :exc:`UndefinedError` 或
        :exc:`SecurityError` 之一。

    .. method:: _fail_with_undefined_error(\*args, \**kwargs)

        参数任意，调用这个方法时会抛出带有由未定义对象上存储的未定义
        hint 生成的错误信息的 :attr:`_undefined_exception` 异常。

.. autoclass:: jinja2.DebugUndefined()

.. autoclass:: jinja2.StrictUndefined()

未定义对象由调用 :attr:`undefined` 创建。

.. admonition:: 实现

    :class:`Undefined` 对象通过重载特殊的 `__underscore__` 方法实现。例如
    默认的 :class:`Undefined` 类实现 `__unicode__` 为返回一个空字符串，但
    `__int__` 和其它会始终抛出异常。你可以自己通过返回 ``0`` 实现转换为
    int::

        class NullUndefined(Undefined):
            def __int__(self):
                return 0
            def __float__(self):
                return 0.0

    要禁用一个方法，重载它并抛出 :attr:`~Undefined._undefined_exception` 。因
    为这在未定义对象中非常常用，未定义对象有辅助方法
    :meth:`~Undefined._fail_with_undefined_error` 自动抛出错误。这里的一个类
    工作类似正规的 :class:`Undefined` ，但它在迭代时阻塞:

        class NonIterableUndefined(Undefined):
            __iter__ = Undefined._fail_with_undefined_error


上下文
-----------

.. autoclass:: jinja2.runtime.Context()
    :members: resolve, get_exported, get_all

    .. attribute:: parent

        一个模板查找的只读全局变量的词典。这些变量可能来自另一个
        :class:`Context` ，或是 :attr:`Environment.globals` ，或是
        :attr:`Template.globals` ，或指向一个由全局变量和传递到渲染函数的变
        量联立的字典。它一定不能被修改。

    .. attribute:: vars

        模板局域变量。这个列表包含环境和来自 :attr:`parent` 范围的上下文函数
        以及局域修改和从模板中导出的变量。模板会在模板求值时修改这个字典，
        但过滤器和上下文函数不允许修改它。

    .. attribute:: environment

        加载该模板的环境

    .. attribute:: exported_vars

        这设定了所有模板导出量的名称。名称对应的值在 :attr:`vars` 字典中。
        可以用 :meth:`get_exported` 获取一份导出变量的拷贝字典。

    .. attribute:: name

        拥有此上下文的模板的载入名。

    .. attribute:: blocks

        模板中块当前映射的字典。字典中的键是块名称，值是注册的块的列表。每个
        列表的最后一项是当前活动的块（继承链中最新的）。

    .. attribute:: eval_ctx

        当前的 :ref:`eval-context` 。

    .. automethod:: jinja2.runtime.Context.call(callable, \*args, \**kwargs)


.. admonition:: 实现

    Python frame 中的局域变量在函数中是不可变的，出于同样的原因，上下文是不可
    变的。 Jinja2 和 Python 都不把上下文/ frame 作为变量的数据存储，而只作为
    主要的数据源。

    当模板访问一个模板中没有定义的变量时， Jinja2 在上下文中查找变量，此后，
    这个变量被视为其是在模板中定义得一样。


.. _loaders:

加载器
-------

加载器负责从诸如文件系统的资源加载模板。环境会把编译的模块像
Python 的 `sys.modules` 一样保持在内存中。与 `sys.models` 不同，无论如何这个
缓存默认有大小限制，且模板会自动重新加载。
所有的加载器都是 :class:`BaseLoader` 的子类。如果你想要创建自己的加载器，继
承 :class:`BaseLoader` 并重载 `get_source` 。

.. autoclass:: jinja2.BaseLoader
    :members: get_source, load

这里有一个 Jinja2 提供的内置加载器的列表:

.. autoclass:: jinja2.FileSystemLoader

.. autoclass:: jinja2.PackageLoader

.. autoclass:: jinja2.DictLoader

.. autoclass:: jinja2.FunctionLoader

.. autoclass:: jinja2.PrefixLoader

.. autoclass:: jinja2.ChoiceLoader

.. autoclass:: jinja2.ModuleLoader


.. _bytecode-cache:

字节码缓存
--------------

Jinja 2.1 和更高的版本支持外部字节码缓存。字节码缓存使得在首次使用时把生成的字节码
存储到文件系统或其它位置来避免处理模板。

这在当你有一个在首个应用初始化的 web 应用， Jinja 一次性编译大量模板拖慢应用时尤其
有用。


要使用字节码缓存，把它实例化并传给 :class:`Environment` 。

.. autoclass:: jinja2.BytecodeCache
    :members: load_bytecode, dump_bytecode, clear

.. autoclass:: jinja2.bccache.Bucket
    :members: write_bytecode, load_bytecode, bytecode_from_string,
              bytecode_to_string, reset

    .. attribute:: environment

        创建 bucket 的 :class:`Environment`

    .. attribute:: key

        该 bucket 的唯一键

    .. attribute:: code

        如果已加载，则为字节码，否则为 `None` 。


内建的字节码缓存:

.. autoclass:: jinja2.FileSystemBytecodeCache

.. autoclass:: jinja2.MemcachedBytecodeCache


实用工具
---------

这些辅助函数和类在你向 Jinja2 环境中添加自定义过滤器或函数时很有用。

.. autofunction:: jinja2.environmentfilter

.. autofunction:: jinja2.contextfilter

.. autofunction:: jinja2.evalcontextfilter

.. autofunction:: jinja2.environmentfunction

.. autofunction:: jinja2.contextfunction

.. autofunction:: jinja2.evalcontextfunction

.. function:: escape(s)

    把字符串 `s` 中 ``&`` 、 ``<`` 、 ``>`` 、 ``'`` 和 ``"`` 转换为 HTML 安
    全的序列。如果你需要在 HTML 中显示可能包含这些字符的文本，可以使用它。这
    个函数不会转义对象。这个函数不会转义含有 HTML 表达式比如已转义数据的对象。

    返回值是一个 :class:`Markup` 字符串。

.. autofunction:: jinja2.clear_caches

.. autofunction:: jinja2.is_undefined

.. autoclass:: jinja2.Markup([string])
    :members: escape, unescape, striptags

.. admonition:: Note

    Jinja2 的 :class:`Markup` 类至少与 Pylons 和 Genshi 兼容。预计不久更多模板
    引擎和框架会采用 `__html__` 的概念。


异常
----------

.. autoexception:: jinja2.TemplateError

.. autoexception:: jinja2.UndefinedError

.. autoexception:: jinja2.TemplateNotFound

.. autoexception:: jinja2.TemplatesNotFound

.. autoexception:: jinja2.TemplateSyntaxError

    .. attribute:: message

        错误信息的 utf-8 字节串。

    .. attribute:: lineno

        发生错误的行号。

    .. attribute:: name

        模板的加载名的 unicode 字符串。

    .. attribute:: filename

        加载的模板的文件名字节串，以文件系统的编码（多是 utf-8 ， Windows
        是 mbcs ）。

    文件名和错误消息是字节串而不是 unicode 字符串的原因是，在 Python 2.x
    中，不对异常和回溯使用 unicode ，编译器同样。这会在 Python 3 改变。
    
.. autoexception:: jinja2.TemplateAssertionError


.. _writing-filters:

自定义过滤器
--------------

自定义过滤器只是常规的 Python 函数，过滤器左边作为第一个参数，其余的参数作
为额外的参数或关键字参数传递到过滤器。

例如在过滤器 ``{{ 42|myfilter(23) }}`` 中，函数被以 ``myfilter(42, 23)`` 调
用。这里给出一个简单的过滤器示例，可以应用到 datetime 对象来格式化它们::

    def datetimeformat(value, format='%H:%M / %d-%m-%Y'):
        return value.strftime(format)

你可以更新环境上的 :attr:`~Environment.filters` 字典来把它注册到模板环境上::

    environment.filters['datetimeformat'] = datetimeformat

在模板中使用如下:

.. sourcecode:: jinja

    written on: {{ article.pub_date|datetimeformat }}
    publication date: {{ article.pub_date|datetimeformat('%d-%m-%Y') }}

也可以传给过滤器当前模板上下文或环境。当过滤器要返回一个未定义值或检查当前的
:attr:`~Environment.autoescape` 设置时很有用。为此，有三个装饰器：
:func:`environmentfilter` 、 :func:`contextfilter` 和
:func:`evalcontextfilter` 。

这里是一个小例子，过滤器把一个文本在 HTML 中换行或分段，并标记返回值为安全
的 HTML 字符串，因为自动转义是启用的::

    import re
    from jinja2 import evalcontextfilter, Markup, escape

    _paragraph_re = re.compile(r'(?:\r\n|\r|\n){2,}')

    @evalcontextfilter
    def nl2br(eval_ctx, value):
        result = u'\n\n'.join(u'<p>%s</p>' % p.replace('\n', '<br>\n')
                              for p in _paragraph_re.split(escape(value)))
        if eval_ctx.autoescape:
            result = Markup(result)
        return result

上下文过滤器工作方式相同，只是第一个参数是当前活动的 :class:`Context` 而
不是环境。


.. _eval-context:

求值上下文
------------------

求值上下文（缩写为 eval context 或 eval ctx ）是 Jinja 2.4 中引入的新对象，
并可以在运行时激活/停用已编译的特性。

当前它只用于启用和禁用自动转义，但也可以用于扩展。

在之前的 Jinja 版本中，过滤器和函数被标记为环境可调用的来从环境中检查自动
转义的状态。在新版本中鼓励通过求值上下文来检查这个设定。

之前的版本::

    @environmentfilter
    def filter(env, value):
        result = do_something(value)
        if env.autoescape:
            result = Markup(result)
        return result

在新版本中，你可以用 :func:`contextfilter` 从实际的上下文中访问求值上下
文，或用 :func:`evalcontextfilter` 直接把求值上下文传递给函数::

    @contextfilter
    def filter(context, value):
        result = do_something(value)
        if context.eval_ctx.autoescape:
            result = Markup(result)
        return result

    @evalcontextfilter
    def filter(eval_ctx, value):
        result = do_something(value)
        if eval_ctx.autoescape:
            result = Markup(result)
        return result

求值上下文一定不能在运行时修改。修改只能在扩展中的
用 :class:`nodes.EvalContextModifier` 和
:class:`nodes.ScopedEvalContextModifier` 发生，而不是通过求值上下文对
象本身。

.. autoclass:: jinja2.nodes.EvalContext

   .. attribute:: autoescape

      `True` 或 `False` 取决于自动转义是否激活。

   .. attribute:: volatile

      如果编译器不能在编译期求出某些表达式的值，为 `True` 。在运行时应该
      始终为 `False` 。


.. _writing-tests:

自定义测试
------------

测试像过滤器一样工作，只是测试不能访问环境或上下文，并且它们不能链式使用。
测试的返回值应该是 `True` 或 `False` 。测试的用途是让模板设计者运行类型和
一致性检查。

这里是一个简单的测试，检验一个变量是否是素数::

    import math

    def is_prime(n):
        if n == 2:
            return True
        for i in xrange(2, int(math.ceil(math.sqrt(n))) + 1):
            if n % i == 0:
                return False
        return True
        

你可以通过更新环境上的 :attr:`~Environment.tests` 字典来注册它::

    environment.tests['prime'] = is_prime

模板设计者可以在之后这样使用测试:

.. sourcecode:: jinja

    {% if 42 is prime %}
        42 is a prime number
    {% else %}
        42 is not a prime number
    {% endif %}


.. _global-namespace:

全局命名空间
--------------------

:attr:`Environment.globals` 字典中的变量是特殊的，它们对导入的模板也是可用的，
即使它们不通过上下文导入。这是你可以放置始终可访问的变量和函数的地方。此外，
:attr:`Template.globals` 是那些对特定模板可用的变量，即对所有的
:meth:`~Template.render` 调用可用。


.. _low-level-api:

低层 API
-------------

低层 API 暴露的功能对理解一些实现细节、调试目的或高级
:ref:`扩展 <jinja-extensions>` 技巧是有用的。除非你准确地了解你在做什么，否则
不推荐使用这些 API 。


.. automethod:: Environment.lex

.. automethod:: Environment.parse

.. automethod:: Environment.preprocess

.. automethod:: Template.new_context

.. method:: Template.root_render_func(context)

    这是低层的渲染函数。它接受一个必须由相同模板或兼容的模板的
    :meth:`new_context` 创建的 :class:`Context` 。这个渲染函数由编译器从
    模板代码产生，并返回一个生产 unicode 字符串的生成器。

    如果模板代码中发生了异常，模板引擎不会重写异常而是直接传递原始的异常。
    事实上，这个函数只在 :meth:`render` / :meth:`generate` / :meth:`stream`
    的调用里被调用。

.. attribute:: Template.blocks

    一个块渲染函数的字典。其中的每个函数与 :meth:`root_render_func` 的工作
    相同，并且有相同的限制。

.. attribute:: Template.is_up_to_date

    如果有可用的新版本模板，这个属性是 `False` ，否则是 `True` 。

.. admonition:: 注意

    低层 API 是易碎的。未来的 Jinja2 的版本将不会试图以不向后兼容的方式修改它，
    而是在 Jinja2 核心的修改中表现出来。比如如果 Jinja2 在之后的版本中引入一
    个新的 AST 节点，它会由 :meth:`~Environment.parse` 返回。

元 API
------------

.. versionadded:: 2.2

元 API 返回一些关于抽象语法树的信息，这些信息能帮助应用实现更多的高级模板概
念。所有的元 API 函数操作一个 :meth:`Environment.parse` 方法返回的抽象语法
树。

.. autofunction:: jinja2.meta.find_undeclared_variables

.. autofunction:: jinja2.meta.find_referenced_templates
