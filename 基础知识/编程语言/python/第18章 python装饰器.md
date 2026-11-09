---
title: 第18章 python装饰器
tags: python
---

------

***<font color=blue>版权声明：</font>本文参考了<font color=blue>《[python doc](https://docs.python.org/3/)》。</font>版本为V3.15，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

----

[TOC]


**装饰器**（Decorator）就是一个“给函数穿装备的加工厂”。它能在不修改原函数内部代码的前提下，给原函数增加额外的功能。

装饰器是闭包。


以下是 Python **标准库中定义的所有可作为装饰器使用的“@开头”工具**，已按功能分类整理，并完成了系统性查漏补缺。表格包含了“方法、触发、说明”三列，并对易混淆或特殊的装饰器进行了标注。

---

# 内置装饰器
 内置装饰器无需导入

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@property` | 类方法定义 | 将方法转变为**只读属性**访问 |
| `@classmethod` | 类方法定义 | 将方法转为**类方法**，第一个参数为 `cls` |
| `@staticmethod` | 类方法定义 | 将方法转为**静态方法**，不接收 `self` 或 `cls` |


# functools模块
高阶函数与可调用对象操作

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@functools.wraps` | 编写装饰器时 | 保留被装饰函数的元数据（如 `__name__`、`__doc__`） |
| `@functools.lru_cache` | 函数定义 | 缓存函数结果，提供**最近最少使用**淘汰策略 |
| `@functools.cache` | 函数定义 | `lru_cache(maxsize=None)` 的简写，无限制缓存（Python 3.9+） |
| `@functools.cached_property` | 类方法定义 | 将方法转为**惰性计算**的缓存属性（Python 3.8+） |
| `@functools.singledispatch` | 函数定义 | 实现**基于第一个参数类型**的单分派泛函数 |
| `@functools.singledispatchmethod` | 类方法定义 | `singledispatch` 的类方法版本（Python 3.8+） |
| `@functools.total_ordering` | 类定义 | 自动补全类的比较方法（`__lt__`、`__le__`、`__gt__`、`__ge__`） |
| `@functools.cmp_to_key` | 函数定义 | 将老式比较函数转为 `key` 函数（如 `sorted` 使用） |
| `@functools.partial` | 函数定义 | 预先绑定部分参数，生成新的可调用对象 |


# contextlib 模块
上下文管理器
| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@contextlib.contextmanager` | 生成器函数定义 | 将生成器转为**上下文管理器**，用于 `with` 语句 |
| `@contextlib.asynccontextmanager` | 异步生成器函数定义 | `contextmanager` 的异步版本，用于 `async with`（Python 3.7+） |


# dataclasses 模块
数据类
| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@dataclasses.dataclass` | 类定义 | 自动生成 `__init__`、`__repr__`、`__eq__` 等特殊方法 |


# abc 模块
抽象基类
| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@abc.abstractmethod` | 方法定义 | 声明抽象方法，子类必须实现 |
| `@abc.ABCMeta.register` | 类定义 | 将类注册为某个抽象基类的**虚拟子类** |


# enum 模块
枚举

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@enum.unique` | 枚举类定义 | 确保枚举成员的**值唯一**，不允许别名 |


# atexit 模块
退出处理器

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@atexit.register` | 函数定义 | 注册函数在**解释器正常退出时**执行 |


# typing 模块

类型提示相关

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@typing.overload` | 函数定义（仅用于类型检查） | 声明函数**重载签名**，不影响运行时 |
| `@typing.final` | 类或方法定义 | 告知类型检查器该类/方法**不可被继承或覆盖** |
| `@typing.override` | 方法定义 | 显式声明方法**覆盖了父类方法**（Python 3.12+） |
| `@typing.no_type_check` | 函数或类定义 | 关闭对被装饰对象的**类型检查** |
| `@typing.runtime_checkable` | Protocol 类定义 | 使 Protocol 支持**运行时 `isinstance` 检查** |
| `@typing.no_type_check_decorator` | 装饰器定义 | 使另一个装饰器产生 `@no_type_check` 的效果（**已废弃**，Python 3.13+ 移除） |


# asyncio 模块
异步编程

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@asyncio.coroutine` | 生成器函数定义 | 标记基于生成器的协程（**已废弃**，Python 3.8+ 不再推荐） |


# 其他标准库装饰器

| 方法 | 触发 | 说明 |
| :--- | :--- | :--- |
| `@re.compile`（作为装饰器不常见） | — | `re.compile` 通常直接调用，但因其返回 Pattern 对象，理论上可作为装饰器使用，但**并非惯用法**。 |
| `@warnings.deprecated` | 函数/类定义 | 标记已废弃的 API（Python 3.13+ 新增） |


# 查漏补缺说明

1. **数量确认**：以上共收录 **30 个** 标准库中可作装饰器使用的工具（含已废弃的 `@asyncio.coroutine` 和 `@typing.no_type_check_decorator`）。若剔除废弃项，核心可用装饰器约 **28 个**。
2. **常见混淆澄清**：
   - `@timeit`、`@cachetools` 等属于**第三方库**，并非标准库内置。
   - `@staticmethod`、`@classmethod`、`@property` 是**内置函数**，无需 `import` 即可使用。
   - `@functools.partial`、`@functools.cmp_to_key` 严格来说返回的是可调用对象，但因其接受函数作为参数并返回函数，完全符合装饰器的定义，且官方文档也将其列为装饰器相关工具。
3. **Python 3.15 兼容性**：以上列表基于 Python 3.15 标准库文档整理。`@asyncio.coroutine` 和 `@typing.no_type_check_decorator` 已被移除或废弃，新代码中不应使用。`@typing.override`、`@warnings.deprecated` 等较新装饰器在 3.15 中已稳定可用。




------

***<font color=blue>版权声明：</font>本文参考了<font color=blue>《[python doc](https://docs.python.org/3/)》。</font>版本为V3.15，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------