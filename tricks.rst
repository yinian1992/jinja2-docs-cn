提示和技巧
===============

.. highlight:: html+jinja

这部分文档展示了一些 Jinja2 模板的提示和技巧。

.. _null-master-fallback:

Null-Master 退回
--------------------

Jinja2 支持动态继承并且只要没有 `extends` 标签被访问过，就不分辨父模板和子模
板。而这会导致令人惊讶的行为：首个 `extends` 标签前的包括空白字符的所有东西
会被打印出来而不是被忽略，这也可以用作一个巧妙的方法。

通常，继承一个模板的子模板来添加基本的 HTML 骨架。而把 `extends` 标签放在
`if` 标签中，当 `standalone` 变量值为 false 时（按照默认未定义也为 false ）继
承布局模板是可行的。此外，一个非常基本的骨架会被添加到文件，这样如果确实带
置为 `True` 的 `standalone` 渲染，一个非常基本的 HTML 骨架会被添加::

    {% if not standalone %}{% extends 'master.html' %}{% endif -%}
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <title>{% block title %}The Page Title{% endblock %}</title>
    <link rel="stylesheet" href="style.css" type="text/css">
    {% block body %}
      <p>This is the page body.</p>
    {% endblock %}


交替的行
----------------

如果你想要对一个表格或列表中的每行使用不同的样式，可以使用 `loop`
对象的 `cycle` 方法::

    <ul>
    {% for row in rows %}
      <li class="{{ loop.cycle('odd', 'even') }}">{{ row }}</li>
    {% endfor %}
    </ul>

`cycle` 可接受无限数目的字符串。每次遭遇这个标签，列表中的下一项
就会被渲染。

高亮活动菜单项
------------------------------

你经常想要一个带有活动导航项的导航栏。这相当容易实现。因为在 `block` 外
的声明在子模板中是全局的，并且在布局模板求值前执行，在子模板中定义活动的
菜单项::

    {% extends "layout.html" %}
    {% set active_page = "index" %}

布局模板之后就可以访问 `active_page` 。此外，这意味着你可以为它定义默认
值::

    {% set navigation_bar = [
        ('/', 'index', 'Index'),
        ('/downloads/', 'downloads', 'Downloads'),
        ('/about/', 'about', 'About')
    ] -%}
    {% set active_page = active_page|default('index') -%}
    ...
    <ul id="navigation">
    {% for href, id, caption in navigation_bar %}
      <li{% if id == active_page %} class="active"{% endif
      %}><a href="{{ href|e }}">{{ caption|e }}</a>/li>
    {% endfor %}
    </ul>
    ...

.. _accessing-the-parent-loop:

访问父级循环
-------------------------

特殊的 `loop` 变量总是指向最里层的循环。如果想要访问外层的循环，可以给它
设置别名::

    <table>
    {% for row in table %}
      <tr>
      {% set rowloop = loop %}
      {% for cell in row %}
        <td id="cell-{{ rowloop.index }}-{{ loop.index }}>{{ cell }}</td>
      {% endfor %}
      </tr>
    {% endfor %}
    </table>
