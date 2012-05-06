沙箱
=======

Jinja2 沙箱用于为不信任的代码求值。访问不安全的属性和方法是被禁止的。

假定在默认配置中 `env` 是一个 :class:`SandboxedEnvironment` 实例，下面的代码展示
了它如何工作:

>>> env.from_string("{{ func.func_code }}").render(func=lambda:None)
u''
>>> env.from_string("{{ func.func_code.do_something }}").render(func=lambda:None)
Traceback (most recent call last):
  ...
SecurityError: access to attribute 'func_code' of 'function' object is unsafe.

API
---

.. module:: jinja2.sandbox

.. autoclass:: SandboxedEnvironment([options])
    :members: is_safe_attribute, is_safe_callable, default_binop_table,
              default_unop_table, intercepted_binops, intercepted_unops,
              call_binop, call_unop

.. autoclass:: ImmutableSandboxedEnvironment([options])

.. autoexception:: SecurityError

.. autofunction:: unsafe

.. autofunction:: is_internal_attribute

.. autofunction:: modifies_known_mutable

.. admonition:: 提示

    Jinja2 沙箱自己并没有彻底解决安全问题。特别是对 web 应用，你必须晓得用户
    可能用任意 HTML 来创建模板，所以保证他们不通过注入 JavaScript 或其它更多
    方法来互相损害至关重要（如果你在同一个服务
    器上运行多用户）。

    同样，沙箱的好处取决于配置。我们强烈建议只向模板传递非共享资源，并
    且使用某种属性白名单。

    也请记住，模板会抛出运行时或编译期错误，确保捕获它们。

运算符拦截
---------------------

.. versionadded:: 2.6

为了性能最大化， Jinja2 会让运算符直接条用类型特定的回调方法。这意味着，
通过重载 :meth:`Environment.call` 来拦截是不可能的。此外，由于运算符的工作
方式，把运算符转换为特殊方法不总是直接可行的。比如为了分类，至少一个特殊
方法存在。

在 Jinja 2.6 中，开始支持显式的运算符拦截。必要时也可以用于自定义的特定
运算符。为了拦截运算符，需要覆写
:attr:`SandboxedEnvironment.intercepted_binops` 属性。当需要拦截的运算符
被添加到这个集合， Jinja2 会生成调用
:meth:`SandboxedEnvironment.call_binop` 函数的字节码。对于一元运算符，
必须替代地使用 `unary` 属性和方法。


:attr:`SandboxedEnvironment.call_binop` 的默认实现会使用
:attr:`SandboxedEnvironment.binop_table` 来把运算符标号翻译成执行默认
运算符行为的回调。

这个例子展示了幂（ ``**`` ）操作符可以在 Jinja2 中禁用::

    from jinja2.sandbox import SandboxedEnvironment


    class MyEnvironment(SandboxedEnvironment):
        intercepted_binops = frozenset(['**'])

        def call_binop(self, context, operator, left, right):
            if operator == '**':
                return self.undefined('the power operator is unavailable')
            return SandboxedEnvironment.call_binop(self, context,
                                                   operator, left, right)

确保始终调入 super 方法，即使你不拦截这个调用。 Jinja2 内部会调用
这个方法来对表达式求值。
