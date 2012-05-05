欢迎来到 Jinja2
=================

Jinja2 是一个现代的，设计者友好的，仿照 Django 模板的 Python 模板语言。
它速度快，被广泛使用，并且提供了可选的沙箱模板执行环境保证安全:

.. sourcecode:: html+jinja

   <title>{% block title %}{% endblock %}</title>
   <ul>
   {% for user in users %}
     <li><a href="{{ user.url }}">{{ user.username }}</a></li>
   {% endfor %}
   </ul>

**特性:**

-   沙箱中执行
-   强大的 HTML 自动转义系统保护系统免受 XSS
-   模板继承
-   及时编译最优的 python 代码
-   可选提前编译模板的时间
-   易于调试。异常的行数直接指向模板中的对应行。
-   可配置的语法

.. include:: contents.rst.inc

如果你找不到你想要的信息，请查看索引或尝试用搜索功能搜寻:

* :ref:`genindex`
* :ref:`search`
