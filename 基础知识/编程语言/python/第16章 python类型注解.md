---
title: 第16章 python类型注解
tags: pyhton


---


# 内置类型

在 Python 中，内置类型不仅是我们日常写代码用的基础数据类型（如整数、字符串），在类型注解的语境下，特指那些**可以直接用来做类型注解，并且在 Python 3.9+ 中支持中括号 `[]` 泛型语法**的原生类。

**核心区别**：
- **内置类型**（如 `list`, `dict`）：用于**具体**的容器。
- **抽象基类**（从 `collections.abc` 导入，如 `Iterable`, `Mapping`）：用于描述**行为/接口**。**官方强烈建议**在标注参数时，优先使用抽象基类（如 `Iterable[int]`），因为它给调用者的限制更少（传 list、tuple、生成器都可以）。

**注意事项**
在函数定义中，`->` 后面的返回类型通常是具体的（比如返回一个列表就写 `list[int]`）。但如果函数是**生成器**，`->` 后面**绝对不能**写内置的容器，必须写 `Iterator` 或 `Generator`（来自 `collections.abc` 或 `typing`），因为生成器不是 `list`。

## 基础标量类型（Scalar Types）
这些是最基础的数据类型，直接写在 `:` 或 `->` 后面即可。

| 内置类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| **`int`** | 整数 | `x: int = 10` |
| **`float`** | 浮点数 | `y: float = 3.14` |
| **`str`** | 字符串 | `name: str = "Alice"` |
| **`bool`** | 布尔值（True/False） | `is_valid: bool = True` |
| **`bytes`** | 字节串 | `data: bytes = b"hello"` |
| **`complex`** | 复数 | `z: complex = 1+2j` |

## 容器/集合类型（支持泛型 `[]` 语法）
**这是 Python 3.9+ 的重大更新**（PEP 585）。你现在可以直接用内置的小写类型加上 `[]` 来做类型注解，完全替代 `typing` 模块里的 `List`, `Dict` 等大写别名。

| 内置类型 | 说明 | 示例（现代写法） | 对应 `typing` 旧写法 |
| :--- | :--- | :--- | :--- |
| **`list`** | 列表 | `list[int]` | `typing.List[int]` |
| **`dict`** | 字典 | `dict[str, int]` | `typing.Dict[str, int]` |
| **`set`** | 集合 | `set[str]` | `typing.Set[str]` |
| **`frozenset`** | 不可变集合 | `frozenset[int]` | `typing.FrozenSet[int]` |
| **`tuple`** | 元组（定长） | `tuple[int, str]` | `typing.Tuple[int, str]` |
| **`tuple`** | 元组（变长） | `tuple[int, ...]` | `typing.Tuple[int, ...]` |

*(注：如果你的代码需要运行在 Python 3.8 及以下，直接写 `list[int]` 会报错。解决办法是在文件顶部加上 `from __future__ import annotations`，或者老老实实换回 `typing.List`。)*


## 特殊内置类型
除了基础数据和容器，Python 还有几个非常特殊的内置类型，在注解中极其常用：

| 内置类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| **`None`** | 空值（其实是 `NoneType` 的单例） | `def f() -> None:` |
| **`type`** | 类对象本身（等价于 `typing.Type`） | `def create(cls: type[MyClass]) -> MyClass:` |
| **`object`** | 所有类的基类（代表“任意对象”，但比 `Any` 安全） | `def process(obj: object) -> None:` |
| **`range`** | 整数范围 | `def loop(r: range) -> None:` |

*关于 `type` 的进阶用法*：在 Python 3.9+ 中，可以直接写 `type[MyClass]` 表示“MyClass 这个类本身”，等价于旧版的 `typing.Type[MyClass]`。


## 内置的异常类型
虽然不常用作业务参数的注解，但在某些场景下（比如写装饰器或异常处理框架）也会用到：
*   `Exception`, `ValueError`, `TypeError` 等。它们本身就是类，可以直接做注解。
    ```python
    def handle_error(err: ValueError) -> None: ...
    ```

# typing模块
为了让你能快速查阅，我将 `typing` 模块中最常用的属性、类和方法整理成了综合表格。

按照功能，我将它们分为了三大类：**用于注解的类型/构造器**、**辅助函数**（用于编写代码或运行时操作），以及**特殊常量**。

*(注：示例基于 Python 3.10+ 语法，部分针对旧版本的兼容写法已省略)*


### 1. 基础与特殊类型（可直接用于 `:` 和 `->`）

| 属性 / 类名 | 作用 | 示例 |
| :--- | :--- | :--- |
| **`Any`** | 任意类型，放弃类型检查 | `def func(x: Any) -> Any: ...` |
| **`Union`** | 联合类型（或） | `x: Union[int, str]` |
| **`Optional`** | 可选类型（等价于 `X \| None`） | `name: Optional[str] = None` |
| **`Callable`** | 可调用对象（如函数、lambda） | `cb: Callable[[int, str], bool]` |
| **`Type`** | 类对象本身（非实例） | `cls: Type[MyClass]` |
| **`Literal`** | 限定具体的字面量值 | `mode: Literal["r", "w", "a"]` |
| **`Final`** | 声明常量，禁止被重新赋值 | `MAX_SIZE: Final = 100` |
| **`ClassVar`** | 类变量（属于类，不属于实例） | `class A: count: ClassVar[int] = 0` |
| **`Self`** | 返回实例自身（3.11+） | `def clone(self) -> Self: ...` |
| **`NoReturn`** | 函数永不正常返回（通常抛异常） | `def fail() -> NoReturn: raise Error()` |
| **`Annotated`** | 附加元数据（供框架使用） | `x: Annotated[int, "positive"]` |

---

### 2. 容器与抽象基类（可直接用于 `:` 和 `->`）
*注：Python 3.9+ 官方推荐使用内置的 `list[int]`, `dict[str, int]`，但 `typing` 的别名依然可用。*

| 属性 / 类名 | 作用 | 示例 |
| :--- | :--- | :--- |
| **`List` / `Dict` / `Set`** | 容器泛型别名 | `x: List[int]`, `y: Dict[str, int]` |
| **`Iterable`** | 可迭代对象（支持 for 循环） | `def f(items: Iterable[int]) -> None: ...` |
| **`Iterator`** | 迭代器（支持 `next()`，一次性） | `def f() -> Iterator[int]: yield 1` |
| **`Sequence`** | 序列（支持索引、切片、长度） | `def f(seq: Sequence[str]) -> str: ...` |
| **`Mapping`** | 映射/字典的抽象基类 | `def f(data: Mapping[str, Any]) -> None: ...` |
| **`Generator`** | 生成器对象 | `def f() -> Generator[int, None, None]: ...` |
| **`Coroutine`** | 协程对象 | `async def f() -> Coroutine[Any, Any, int]: ...` |

---

### 3. 泛型与特殊构造器

| 属性 / 类名 | 作用 | 示例 |
| :--- | :--- | :--- |
| **`TypeVar`** | 定义泛型类型变量 | `T = TypeVar('T')`<br>`def f(x: T) -> T: return x` |
| **`Generic`** | 自定义泛型类的基类 | `class Box(Generic[T]): ...` |
| **`Protocol`** | 鸭子类型协议（结构化子类型） | `class Drawable(Protocol): def draw(self): ...` |
| **`TypedDict`** | 定义字典的键值结构 | `class User(TypedDict): name: str` |
| **`NamedTuple`** | 创建带字段名的元组 | `class Point(NamedTuple): x: int; y: int` |
| **`NewType`** | 创建逻辑上独立的子类型 | `UserId = NewType('UserId', int)` |

---

### 4. 辅助函数（用于逻辑判断、代码编写、运行时获取）

*这些函数不能直接作为类型写在 `:` 或 `->` 后面。*

| 函数名 | 作用 | 示例 |
| :--- | :--- | :--- |
| **`cast()`** | 强制告诉类型检查器把某值当作某类型 | `x = cast(int, value)` |
| **`overload()`** | 定义函数的重载签名 | `@overload\ndef f(x: int) -> int: ...` |
| **`final()`** | 装饰器，表示类/方法不可继承或覆盖 | `@final\nclass Base: ...` |
| **`runtime_checkable()`** | 让 `Protocol` 支持运行时 `isinstance` 检查 | `@runtime_checkable\nclass P(Protocol): ...` |
| **`get_type_hints()`** | 运行时获取函数的真实类型注解 | `get_type_hints(func)` |
| **`get_args()` / `get_origin()`** | 内省泛型类型的具体参数 | `get_args(List[int]) # 返回 (int,)` |
| **`dataclass_transform()`** | 声明某个装饰器具有 `@dataclass` 的特性 | `@dataclass_transform()\ndef my_decorator(cls): ...` |
| **`reveal_type()`** | 调试用，让 IDE 打印出推断出的类型 | `reveal_type(x) # IDE 提示: int` |
| **`assert_type()`** | 断言某个变量就是某个类型 | `assert_type(x, int)` |

---

### 5. 特殊常量

| 常量名 | 作用 | 示例 |
| :--- | :--- | :--- |
| **`TYPE_CHECKING`** | 布尔值（运行时为 `False`，检查时为 `True`） | `if TYPE_CHECKING: from x import Y` |

---

### 💡 补充提醒
*   **方法**：`typing` 模块主要提供的是**类和函数**，并没有类似 `list.append()` 那种绑定在实例上的“方法”。上面的 `get_args()`、`cast()` 等都是模块级别的函数。
*   **版本差异**：像 `Self`、`TypeGuard`、`Unpack` 等是较新版本（3.10/3.11+）引入的，如果你在老版本 Python 中使用，Pylance 会提示你从 `typing_extensions` 库中导入。