---
title: 第16章 python魔法方法
tags: python
---


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[python doc](https://docs.python.org/3/)》。</font>版本为V3.14.3，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

[TOC]


**魔法方法是 Python 内置的一种“暗号”。当你对对象使用某些特定的内置语法（如加号、索引、打印、循环）时，Python 会偷偷地去调用这些“暗号”对应的方法。**

Python 的魔法方法（Magic Methods，又称 Dunder Methods）大约有 **80-100 个**，它们是让自定义对象能像内置类型一样融入 Python 语法体系的核心。

下面是按功能分类的完整速查表，覆盖了绝大多数常用和进阶的魔法方法。


# 生命周期与创建

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__new__(cls, ...)` | `obj = MyClass()` | 真正的构造函数，负责**创建并返回**实例，在 `__init__` 之前调用 |
| `__init__(self, ...)` | `obj = MyClass()` | 初始化方法，负责设置实例属性，在对象创建后调用 |
| `__del__(self)` | `del obj` 或垃圾回收时 | 析构器，对象被销毁时调用（**不保证一定执行**） |

# 字符串表示与格式化

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__str__(self)` | `str(obj)`、`print(obj)` | 返回**面向用户**的字符串表示，可读性优先 |
| `__repr__(self)` | `repr(obj)`、交互式显示 | 返回**面向开发者**的字符串表示，理想情况下应能重建对象 |
| `__format__(self, spec)` | `f"{obj:spec}"`、`format(obj, spec)` | 自定义格式化规则，用于 f-string 或 `format()` 函数 |
| `__bytes__(self)` | `bytes(obj)` | 定义对象的字节表示，返回 `bytes` 对象 |


# 比较运算

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__eq__(self, other)` | `a == b` | 等于，定义相等性判断 |
| `__ne__(self, other)` | `a != b` | 不等于，通常可省略（Python 会自动反转 `__eq__` 的结果） |
| `__lt__(self, other)` | `a < b` | 小于 |
| `__le__(self, other)` | `a <= b` | 小于等于 |
| `__gt__(self, other)` | `a > b` | 大于 |
| `__ge__(self, other)` | `a >= b` | 大于等于 |
| `__hash__(self)` | `hash(obj)`、作为 `dict` 键或 `set` 成员 | 定义对象的哈希值。**重写 `__eq__` 时必须同时重写 `__hash__`**，以保持一致性 |
| `__bool__(self)` | `bool(obj)`、`if obj:` | 定义对象的真假值判断 |

---

# 算术运算（普通、反向、就地）

Python 的算术魔法方法有三种变体：**普通**（`a + b`）、**反向**（当左操作数不支持时，如 `b + a` 会尝试 `a.__radd__(b)`）、**就地**（`a += b`，通常返回修改后的 `self`）。

| 运算 | 普通方法 | 反向方法 | 就地方法 |
| :--- | :--- | :--- | :--- |
| **`+`** | `__add__` | `__radd__` | `__iadd__` |
| **`-`** | `__sub__` | `__rsub__` | `__isub__` |
| **`*`** | `__mul__` | `__rmul__` | `__imul__` |
| **`/`** | `__truediv__` | `__rtruediv__` | `__itruediv__` |
| **`//`** | `__floordiv__` | `__rfloordiv__` | `__ifloordiv__` |
| **`%`** | `__mod__` | `__rmod__` | `__imod__` |
| **`**`** | `__pow__` | `__rpow__` | `__ipow__` |
| **`@`** | `__matmul__` | `__rmatmul__` | `__imatmul__` |
| **`&`** | `__and__` | `__rand__` | `__iand__` |
| **`|`** | `__or__` | `__ror__` | `__ior__` |
| **`^`** | `__xor__` | `__rxor__` | `__ixor__` |
| **`<<`** | `__lshift__` | `__rlshift__` | `__ilshift__` |
| **`>>`** | `__rshift__` | `__rrshift__` | `__irshift__` |

# 其他数学运算

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__neg__(self)` | `-a` | 取负 |
| `__pos__(self)` | `+a` | 取正 |
| `__abs__(self)` | `abs(a)` | 绝对值 |
| `__divmod__(self, other)` | `divmod(a, b)` | 返回 `(a // b, a % b)` |
| `__invert__(self)` | `~a` | 按位取反（通常配合 `__index__`） |

# 容器与序列模拟

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__len__(self)` | `len(obj)` | 返回容器元素个数，必须是 ≥0 的整数 |
| `__getitem__(self, key)` | `obj[key]` | 获取元素，支持整数索引和切片对象 |
| `__setitem__(self, key, value)` | `obj[key] = value` | 设置元素 |
| `__delitem__(self, key)` | `del obj[key]` | 删除元素 |
| `__contains__(self, item)` | `item in obj` | 成员判断（`in` / `not in`） |
| `__iter__(self)` | `for x in obj:` | 返回一个迭代器，使对象可被遍历 |
| `__next__(self)` | `next(obj)` | 返回迭代器的下一个值，无值时抛出 `StopIteration` |
| `__reversed__(self)` | `reversed(obj)` | 返回一个反向迭代器 |

# 可调用对象

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__call__(self, *args, **kwargs)` | `obj(args)` | 让**实例**可以像函数一样被调用 |

# 上下文管理器

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__enter__(self)` | `with obj as x:` 的进入阶段 | 返回值会赋给 `as` 后的变量 |
| `__exit__(self, exc_type, exc_val, tb)` | `with` 块结束或异常时 | 负责清理。返回 `True` 可**抑制异常**，否则异常会继续传播 |



# 属性访问与描述符

| 魔法方法 | 触发方式 | 说明 |
| :--- | :--- | :--- |
| `__getattr__(self, name)` | `obj.attr`（**仅当常规查找失败时**） | 作为属性查找的**后备机制**，处理“找不到”的情况 |
| `__getattribute__(self, name)` | `obj.attr`（**所有**属性访问） | 拦截**每一次**属性访问，**使用需极其谨慎**，易造成无限递归 |
| `__setattr__(self, name, value)` | `obj.attr = value` | 拦截所有属性赋值操作 |
| `__delattr__(self, name)` | `del obj.attr` | 拦截属性删除操作 |
| `__dir__(self)` | `dir(obj)` | 自定义 `dir()` 的返回结果 |
| `__get__(self, instance, owner)` | 描述符：读取属性时 | 定义**描述符**（Descriptor）的读取行为 |
| `__set__(self, instance, value)` | 描述符：设置属性时 | 定义**数据描述符**的写入行为（有 `__set__` 即成为数据描述符） |
| `__delete__(self, instance)` | 描述符：删除属性时 | 定义描述符的删除行为 |
| `__set_name__(self, owner, name)` | 类创建时 | 在类定义时自动调用，告知描述符它被赋予的属性名 |



# 使用要点

1. **`__repr__` 优先**：如果只定义 `__repr__`，`str()` 会自动调用它作为后备。
2. **`__eq__` 与 `__hash__` 联动**：定义了 `__eq__` 而没定义 `__hash__` 时，对象会变得**不可哈希**（不能做 `dict` 键或放入 `set`）。
3. **返回 `NotImplemented`**：在 `__eq__` 等比较方法中，如果无法处理另一类型，应返回 `NotImplemented`（而非 `False`），以允许 Python 尝试对方的反向比较方法。
4. **优先使用 `__str__` 和 `__repr__`**：日常开发中，这两个是最常实现的，用于调试和日志输出。

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[python doc](https://docs.python.org/3/)》。</font>版本为V3.14.3，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------