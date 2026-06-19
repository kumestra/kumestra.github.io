---
title: "Python Type Hints 可以写在哪里"
description: "系统梳理 Python 中可以添加类型提示的主要场景，并为每一种方式提供代码示例。"
pubDatetime: 2026-06-19T23:54:10Z
tags: [python, typing, type-hints, static-typing]
---

# Python Type Hints 可以写在哪里

Python 仍然是一门动态类型语言。类型提示写在源码里，但默认不会在运行时强制检查。它们的主要用途是帮助类型检查器、IDE、Linter、文档工具和框架理解代码意图。

一个实用的心智模型是：Type Hints 是写给“人和工具”的类型契约，而不是 Python 解释器默认强制执行的运行时规则。🧭

## 最基础的三类位置

| 场景 | 示例 | 含义 |
| --- | --- | --- |
| 变量 | `name: str = "Alice"` | 变量应当是字符串 |
| 参数 | `def greet(name: str)` | 参数应当是字符串 |
| 返回值 | `def greet(...) -> str` | 返回值应当是字符串 |

```python
def greet(name: str) -> str:
    return "Hello " + name
```

如果函数没有返回值，通常写 `None`：

```python
def log(message: str) -> None:
    print(message)
```

## 变量注解

### 局部变量

```python
def build_name() -> str:
    name: str = "Alice"
    return name
```

### 模块级变量

```python
APP_NAME: str = "vibe-flow"
MAX_RETRY: int = 3
```

### 只声明类型，不立即赋值

```python
user_name: str
```

这只声明类型意图。如果在赋值前读取变量，仍然会产生运行时错误。

### 常量

```python
from typing import Final

API_VERSION: Final[str] = "v1"
```

`Final` 表示类型检查器应当阻止重新赋值。

## 函数参数

### 普通参数

```python
def add(a: int, b: int) -> int:
    return a + b
```

### 默认参数

```python
def repeat(text: str, count: int = 1) -> str:
    return text * count
```

`count: int = 1` 中，`int` 是类型提示，`1` 是默认值。

### 可选参数

```python
def find_user(user_id: int | None) -> str:
    if user_id is None:
        return "anonymous"

    return str(user_id)
```

旧写法是：

```python
from typing import Optional


def find_user(user_id: Optional[int]) -> str:
    return str(user_id)
```

### 联合类型

```python
def stringify(value: str | int | float) -> str:
    return str(value)
```

旧写法是：

```python
from typing import Union


def stringify(value: Union[str, int, float]) -> str:
    return str(value)
```

### `*args`

```python
def join_all(*items: str) -> str:
    return ",".join(items)
```

这里表示每一个变长位置参数都是 `str`。

### `**kwargs`

```python
def build_user(**fields: str) -> dict[str, str]:
    return fields
```

这里表示每一个关键字参数的值都是 `str`。

## 容器类型

### list

```python
names: list[str] = ["Alice", "Bob"]
```

### dict

```python
scores: dict[str, int] = {
    "Alice": 95,
    "Bob": 88,
}
```

### set

```python
tags: set[str] = {"python", "typing"}
```

### 固定长度 tuple

```python
point: tuple[int, int] = (10, 20)
user: tuple[int, str] = (1, "Alice")
```

### 可变长度 tuple

```python
numbers: tuple[int, ...] = (1, 2, 3, 4)
```

### 空 tuple

```python
empty: tuple[()] = ()
```

## 抽象容器类型

函数通常不需要限制调用者必须传入 `list`。如果只需要“可迭代”或“可索引序列”，可以使用抽象类型。

```python
from collections.abc import Iterable, Iterator, Mapping, Sequence


def consume(items: Iterable[str]) -> None:
    for item in items:
        print(item)


def first(items: Sequence[str]) -> str:
    return items[0]


def read_config(config: Mapping[str, str]) -> str:
    return config["model"]


def get_items() -> Iterator[int]:
    yield 1
    yield 2
```

## Callable

`Callable` 用来描述“可以被调用的对象”。

```python
from collections.abc import Callable


def run_tool(tool: Callable[[], str]) -> str:
    return tool()
```

带参数的函数类型：

```python
from collections.abc import Callable


def apply(fn: Callable[[int, int], int]) -> int:
    return fn(1, 2)
```

任意参数：

```python
from collections.abc import Callable

callback: Callable[..., str]
```

## 异步与生成器

### async 函数

```python
async def fetch_text() -> str:
    return "hello"
```

这里的 `-> str` 表示 `await fetch_text()` 之后得到 `str`。

### Awaitable

```python
from collections.abc import Awaitable


def make_task() -> Awaitable[str]:
    async def inner() -> str:
        return "done"

    return inner()
```

### Generator

```python
from collections.abc import Generator


def counter() -> Generator[int, None, None]:
    yield 1
    yield 2
```

`Generator[YieldType, SendType, ReturnType]` 分别表示 yield 出来的类型、send 进去的类型、return 的类型。

### Iterator

```python
from collections.abc import Iterator


def counter() -> Iterator[int]:
    yield 1
    yield 2
```

### AsyncIterator

```python
from collections.abc import AsyncIterator


async def stream_text() -> AsyncIterator[str]:
    yield "hello"
    yield "world"
```

这类类型特别适合描述 SSE、LLM streaming、异步事件流。

### AsyncGenerator

```python
from collections.abc import AsyncGenerator


async def stream_ids() -> AsyncGenerator[int, None]:
    yield 1
    yield 2
```

### Coroutine

```python
from collections.abc import Coroutine


def get_coroutine() -> Coroutine[None, None, str]:
    async def inner() -> str:
        return "ok"

    return inner()
```

## 类相关类型提示

### 类属性

```python
class Agent:
    model_name: str
    max_rounds: int
```

### 实例属性

```python
class AgentMessage:
    def __init__(self, message_type: str, content: str) -> None:
        self.message_type: str = message_type
        self.content: str = content
```

### 类变量

```python
from typing import ClassVar


class Agent:
    default_model: ClassVar[str] = "gpt-4.1"
```

### `__init__`

```python
class Agent:
    def __init__(self, model_name: str) -> None:
        self.model_name = model_name
```

`__init__` 通常写 `-> None`。

### property

```python
class User:
    def __init__(self, name: str) -> None:
        self._name = name

    @property
    def name(self) -> str:
        return self._name
```

setter 也可以标注：

```python
class User:
    def __init__(self) -> None:
        self._age = 0

    @property
    def age(self) -> int:
        return self._age

    @age.setter
    def age(self, value: int) -> None:
        self._age = value
```

### staticmethod

```python
class Parser:
    @staticmethod
    def parse_int(text: str) -> int:
        return int(text)
```

### classmethod

```python
from typing import Self


class User:
    def __init__(self, name: str) -> None:
        self.name = name

    @classmethod
    def create(cls, name: str) -> Self:
        return cls(name)
```

### `__call__`

```python
class Tool:
    def __call__(self, name: str) -> str:
        return "hello " + name
```

### 普通迭代器协议

```python
from typing import Self


class Counter:
    def __iter__(self) -> Self:
        return self

    def __next__(self) -> int:
        raise StopIteration
```

### 异步迭代器协议

```python
from typing import Self


class AsyncCounter:
    def __aiter__(self) -> Self:
        return self

    async def __anext__(self) -> int:
        raise StopAsyncIteration
```

### 上下文管理器

```python
from types import TracebackType


class Resource:
    def __enter__(self) -> "Resource":
        return self

    def __exit__(
        self,
        exc_type: type[BaseException] | None,
        exc: BaseException | None,
        tb: TracebackType | None,
    ) -> bool:
        return False
```

### 异步上下文管理器

```python
from types import TracebackType


class AsyncResource:
    async def __aenter__(self) -> "AsyncResource":
        return self

    async def __aexit__(
        self,
        exc_type: type[BaseException] | None,
        exc: BaseException | None,
        tb: TracebackType | None,
    ) -> bool:
        return False
```

## 字面量、别名与新类型

### Literal

```python
from typing import Literal

MessageType = Literal["llm_str", "tool_call", "error"]


def emit(message_type: MessageType) -> None:
    print(message_type)
```

`Literal` 表示只能使用指定的字面量值。

### Type Alias

Python 3.12+ 写法：

```python
type UserId = int
type JsonDict = dict[str, object]
```

旧写法：

```python
UserId = int
JsonDict = dict[str, object]
```

显式旧写法：

```python
from typing import TypeAlias

UserId: TypeAlias = int
```

### NewType

```python
from typing import NewType

UserId = NewType("UserId", int)


def get_user_name(user_id: UserId) -> str:
    return str(user_id)
```

`NewType` 用来表达“运行时还是原类型，但静态类型上是不同概念”。

## 结构化字典与数据对象

### TypedDict

```python
from typing import TypedDict


class ToolFunction(TypedDict):
    name: str
    arguments: str
```

使用：

```python
function: ToolFunction = {
    "name": "get_current_time",
    "arguments": "{}",
}
```

### TypedDict 可选字段

```python
from typing import NotRequired, TypedDict


class User(TypedDict):
    id: int
    name: str
    email: NotRequired[str]
```

### TypedDict 必填字段

```python
from typing import Required, TypedDict


class User(TypedDict, total=False):
    id: Required[int]
    name: str
```

### ReadOnly 字段

```python
from typing import ReadOnly, TypedDict


class Config(TypedDict):
    model: ReadOnly[str]
```

### NamedTuple

```python
from typing import NamedTuple


class Point(NamedTuple):
    x: int
    y: int
```

### dataclass

```python
from dataclasses import dataclass


@dataclass
class AgentMessage:
    message_type: str
    message_content: str
```

## Protocol 与结构化子类型

`Protocol` 用来表达“只要行为匹配即可”，不要求继承某个具体类。🔌

```python
from typing import Protocol


class Tool(Protocol):
    def __call__(self) -> str:
        ...


def run_tool(tool: Tool) -> str:
    return tool()
```

## 泛型

### 泛型函数

Python 3.12+ 写法：

```python
from collections.abc import Sequence


def first[T](items: Sequence[T]) -> T:
    return items[0]
```

旧写法：

```python
from collections.abc import Sequence
from typing import TypeVar

T = TypeVar("T")


def first(items: Sequence[T]) -> T:
    return items[0]
```

### 泛型类

Python 3.12+ 写法：

```python
class Box[T]:
    def __init__(self, value: T) -> None:
        self.value = value

    def get(self) -> T:
        return self.value
```

旧写法：

```python
from typing import Generic, TypeVar

T = TypeVar("T")


class Box(Generic[T]):
    def __init__(self, value: T) -> None:
        self.value = value

    def get(self) -> T:
        return self.value
```

### 受约束的 TypeVar

```python
from typing import TypeVar

AnyStr = TypeVar("AnyStr", str, bytes)


def concat(a: AnyStr, b: AnyStr) -> AnyStr:
    return a + b
```

### 有上界的 TypeVar

```python
from typing import TypeVar


class User:
    pass


TUser = TypeVar("TUser", bound=User)


def clone_user(user: TUser) -> TUser:
    return user
```

### Self

```python
from typing import Self


class Builder:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self
```

### 类对象类型 `type[C]`

```python
class User:
    pass


def create_user(user_class: type[User]) -> User:
    return user_class()
```

`type[User]` 表示参数应该是 `User` 类本身，或其子类，而不是 `User` 实例。

### TypeVarTuple 与 Unpack

```python
from typing import TypeVarTuple, Unpack

Ts = TypeVarTuple("Ts")


class Array[*Ts]:
    shape: tuple[Unpack[Ts]]
```

旧风格：

```python
from typing import Generic, TypeVarTuple, Unpack

Ts = TypeVarTuple("Ts")


class Array(Generic[Unpack[Ts]]):
    pass
```

## 装饰器与高阶函数

### ParamSpec

```python
from collections.abc import Callable
from typing import ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")


def log_call(fn: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print("calling")
        return fn(*args, **kwargs)

    return wrapper
```

### Concatenate

```python
from collections.abc import Callable
from typing import Concatenate, ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")


class Request:
    pass


def with_request(
    fn: Callable[Concatenate[Request, P], R],
) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        request = Request()
        return fn(request, *args, **kwargs)

    return wrapper
```

## 元数据、收窄与类型检查辅助

### Annotated

```python
from typing import Annotated

UserName = Annotated[str, "must not be empty"]


def create_user(name: UserName) -> None:
    print(name)
```

`Annotated` 可以在类型之外附加元数据。

### cast

```python
from typing import cast

value: object = "hello"
text = cast(str, value)
```

`cast` 只影响类型检查器，不做运行时转换。

### assert_type

```python
from typing import assert_type

name = "Alice"
assert_type(name, str)
```

### reveal_type

```python
from typing import reveal_type

value = "hello"
reveal_type(value)
```

### TypeGuard

```python
from typing import TypeGuard


def is_str_list(value: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(item, str) for item in value)
```

### TypeIs

```python
from typing import TypeIs


def is_str(value: object) -> TypeIs[str]:
    return isinstance(value, str)
```

### NoReturn

```python
from typing import NoReturn


def fail(message: str) -> NoReturn:
    raise RuntimeError(message)
```

### Never

```python
from typing import Never


def unreachable(value: Never) -> Never:
    raise AssertionError("unreachable")
```

### LiteralString

```python
from typing import LiteralString


def run_query(sql: LiteralString) -> None:
    print(sql)
```

## 前向引用与导入控制

### 字符串前向引用

```python
class Tree:
    def children(self) -> list["Tree"]:
        return []
```

### 推迟注解求值

```python
from __future__ import annotations


class Tree:
    def children(self) -> list[Tree]:
        return []
```

### TYPE_CHECKING

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from expensive_module import Client


def connect(client: "Client") -> None:
    print(client)
```

`TYPE_CHECKING` 在运行时是 `False`，但类型检查器会读取其中的导入。

## overload

```python
from typing import overload


@overload
def get_value(key: str) -> str:
    ...


@overload
def get_value(key: int) -> int:
    ...


def get_value(key: str | int) -> str | int:
    return key
```

`overload` 用来描述同一个函数在不同参数类型下有不同返回类型。

## Stub 文件与旧式类型注释

### `.pyi` stub 文件

```python
# user.pyi
class User:
    name: str

    def login(self, password: str) -> bool:
        ...
```

`.pyi` 常用于给第三方库或 C 扩展提供类型信息。

### `# type:` 注释

```python
name = "Alice"  # type: str
```

函数旧式写法：

```python
def add(a, b):
    # type: (int, int) -> int
    return a + b
```

现代 Python 更推荐直接使用注解语法。

### `# type: ignore`

```python
value = unknown_library_call()  # type: ignore
```

更精确的写法：

```python
value = unknown_library_call()  # type: ignore[no-untyped-call]
```

这不是添加类型，而是告诉类型检查器忽略这一行的错误。

## 框架字段定义

一些框架会读取类型提示，并把它们用于运行时行为。例如数据模型字段：

```python
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name: str
```

这类场景中，类型提示既能帮助静态检查，也可能被框架用于校验、序列化或文档生成。

## 快速检查清单 ✅

写 Python Type Hints 时，可以优先检查这些位置：

- [ ] 函数参数是否有类型
- [ ] 函数返回值是否有类型
- [ ] `__init__` 是否写了 `-> None`
- [ ] 类属性和实例属性是否清楚
- [ ] 容器是否写了元素类型
- [ ] 字典结构是否应该用 `TypedDict`
- [ ] 事件类型、状态值是否应该用 `Literal`
- [ ] 回调函数是否应该用 `Callable` 或 `Protocol`
- [ ] streaming / generator 是否应该用 `Iterator` 或 `AsyncIterator`
- [ ] 装饰器是否需要 `ParamSpec`
- [ ] 复杂签名是否应该抽成 type alias

## 总结

Python Type Hints 可以出现在变量、函数、类、容器、异步流、生成器、装饰器、协议、结构化字典、stub 文件和类型检查辅助代码中。

它的核心价值不是把 Python 变成强制静态类型语言，而是把代码里的“类型意图”显式写出来。这样人更容易读，工具更容易检查，复杂系统也更容易逐层演进。✨
