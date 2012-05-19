从其它的模板引擎切换
=====================================

.. highlight:: html+jinja

如果你过去使用一个不同的模板引擎，并且想要转换到 Jinja2 ，这里是一份简小的
指导展示了一些常见的、相似的 Python 文本模板引擎基本语法和语义差异。

Jinja1
------

Jinja2 与 Jinja1 在 API 使用和模板语法上最为兼容。下面的列表解释了 Jinja1 和
Jinja2 的区别。

API
~~~

加载器
    Jinja2 使用不同的加载器 API 。因为模板的内部表示更改，不再支持 memcached
    这样的外部缓存系统。模板的内存开销与常规的 Python 模块相当，外部缓存不能
    带来优势。如果你以前使用了一个自定义的加载器，请阅读
    :ref:`loader API <loaders>` 部分。

从字符串加载模板
    在过去，在默认环境配置中使用 `jinja.from_string` 从字符串生成模板是可能
    的。 Jinja2 提供了一个 :class:`Template` 类来用于做同样的事情，但是需要
    可选的额外配置。

自动 Unicode 转换
    Jinja1 执行把字节串从一个给定编码到 unicode 对象的自动转换。这个转换不再
    被实现，因为它与大多数使用常规 Python ASCII 字节串到 Unicode 转换的库不
    一致。一个由 Jinja2 驱动的应用 *必须* 在内部的每个地方都使用 unicode 或
    确保 Jinja2 只会被传递 unicode 字符串。

i18n
    Jinja1 使用自定义的国际化翻译器。 i18n 现在作为 Jinja2 的一个扩展，并且
    使用更简单、更 gettext 友好的接口，并且支持 babel 。更多细节见
    :ref:`i18n-extension` 。

内部方法
    Jinja1 在环境对象上暴露了诸如 `call_function` 、 `get_attribute` 等内部
    方法。当它们被标记为一个内部方法，则可以覆盖它们。 Jinja2 并没有等价的
    方法。

沙箱
    Jinja1 默认运行沙箱模式。实际上只有少数应用使用这一特性，所以这在
    Jinja2 中是可选的。更多关于上下执行的细节见
    :class:`SandboxedEnvironment` 。

上下文
    Jinja1 有一个上下文栈存储传递到环境的变量。在 Jinja2 中有一个类似的
    对象，但它不允许修改也不是单例的。由于继承是动态的，现在当模板求值时
    可能存在多个上下文对象。

过滤器和测试
    过滤器和测试现在是常规的函数。不再允许使用工厂函数，且也没有必要。

模板
~~~~~~~~~

Jinja2 与 Jinja1 的语法几乎相同。区别是，现在宏需要用小括号包裹参数。

此外， Jinja2 允许动态继承和动态包含。老的辅助函数 `rendertemplate` 作古，
而使用 `include` 。包含不再导入宏和变量声明，因为采用了新的 `import` 标签。
这个概念在 :ref:`import` 文档中做了解释。

另一个改变发生在 `for` 标签里。特殊的循环变量不再拥有 `parent` 属性，而
你需要自己给循环起别名。见 :ref:`accessing-the-parent-loop` 了解更多细节。

Django
------

如果你之前使用 Django 模板，你应该会发现跟 Jinja2 非常相似。实际上，
很多的语法元素看起来相同，工作也相同。

尽管如此， Jinja2 提供了更多的在之前文档中描述的语法元素，并且某些
工作会有一点不一样。

本节介绍了模板差异。由于 API 是从根本上不同，我们不会再这里赘述。

方法调用
~~~~~~~~~~~~

在 Django 中，方法调用是隐式的。在 Jinja2 中，你必须指定你要调用一个对象。如
此，这段 Django 代码::

    {% for page in user.get_created_pages %}
        ...
    {% endfor %}
    
在 Jinja 中应该是这样::

    {% for page in user.get_created_pages() %}
        ...
    {% endfor %}

这允许你给函数传递变量，且宏也使用这种方式，而这在 Django 中是不可能的。

条件
~~~~~~~~~~

在 Django 中你可以使用下面的结构来判断是否相等::

    {% ifequal foo "bar" %}
        ...
    {% else %}
        ...
    {% endifequal %}

在 Jinja2 中你可以像通常一样使用 if 语句和操作符来做比较::

    {% if foo == 'bar' %}
        ...
    {% else %}
        ...
    {% endif %}

你也可以在模板中使用多个 elif 分支::

    {% if something %}
        ...
    {% elif otherthing %}
        ...
    {% elif foothing %}
        ...
    {% else %}
        ...
    {% endif %}

过滤器参数
~~~~~~~~~~~~~~~~

Jinja2 为过滤器提供不止一个参数。参数传递的语法也是不同的。一个这样的
Django 模板::

    {{ items|join:", " }}

在 Jinja2 中是这样::

    {{ items|join(', ') }}

实际上这有点冗赘，但它允许不同类型的参数——包括变量——且不仅是一种。

测试
~~~~~

除过滤器外，同样有用 is 操作符运行的测试。这里是一些例子::

    {% if user.user_id is odd %}
        {{ user.username|e }} is odd
    {% else %}
        hmm. {{ user.username|e }} looks pretty normal
    {% endif %}

循环
~~~~~

因为循环与 Django 中的十分相似，仅有的不兼容是 Jinja2 中循环上下文的特殊变
量名为 `loop` 而不是 Django 中的 `forloop` 。

周期计
~~~~~~

Jinja 中没有 ``{% cycle %}`` 标签，因为它是隐式的性质。而你可以用循环对象
的 `cycle` 方法实现几乎相同的东西。

下面的 Django 模板::

    {% for user in users %}
        <li class="{% cycle 'odd' 'even' %}">{{ user }}</li>
    {% endfor %}

Jinja 中看起来是这样::

    {% for user in users %}
        <li class="{{ loop.cycle('odd', 'even') }}">{{ user }}</li>
    {% endfor %}

没有与 ``{% cycle ... as variable %}`` 等价的。


Mako
----

.. highlight:: html+mako

如果你迄今使用 Mako 并且想要转换到 Jinja2 ，你可以把 Jinja2 配置成 Mako 一
样:

.. sourcecode:: python

    env = Environment('<%', '%>', '${', '}', '%')

环境配置成这样后， Jinja2 应该可以解释一个 Mako 模板的小型子集。 Jinja2 不支持
嵌入 Python 代码，所以你可能需要把它们移出模板。 def 的语法（在 Jinja2 中 def
被叫做宏）并且模板继承也是不同的。下面的 Mako 模板::

    <%inherit file="layout.html" />
    <%def name="title()">Page Title</%def>
    <ul>
    % for item in list:
        <li>${item}</li>
    % endfor
    </ul>

在以上配置的 Jinja2 中看起来是这样::

    <% extends "layout.html" %>
    <% block title %>Page Title<% endblock %>
    <% block body %>
    <ul>
    % for item in list:
        <li>${item}</li>
    % endfor
    </ul>
    <% endblock %>
