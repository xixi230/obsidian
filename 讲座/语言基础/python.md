相对最简单的语言，也是相对最为适合入门的现代语言
特点，极为简洁的语法和人性化

解释型语言，它用**解释器**和虚拟机，将源代码的编译和执行过程解耦了。



## VScode中配置Python运行环境
1. vscode 的安装
官网：[Visual Studio Code - The open source AI code editor](https://code.visualstudio.com/)
![[Pasted image 20260131104911.png]]
	点击download，就可下载
---
安装页面

![[Pasted image 20260131105022.png]]


	读条完毕就可以安装了
---

2. python解释器安装
官网：[Welcome to Python.org](https://www.python.org/)
![[Pasted image 20260131105856.png]]
下载页面：[Python Releases for Windows | Python.org](https://www.python.org/downloads/windows/)
![[Pasted image 20260131110512.png]]
在这里可以直接安装最新的3.14.2（建议不要安装最新的，因为某些核心依赖包（如 `numpy`、`pandas`、`django` 等）的稳定版可能只标注支持到 Python 3.12.x）
`这是我的`
![[Pasted image 20260131110233.png]]

注意：一定要勾选这个，添加路径
![[Pasted image 20260131110722.png]]
读条完就可以了

3. vscode安装插件
![[Pasted image 20260131111014.png]]

vscode默认英文，所以我们要先安装中文插件（Chinese)，然后再去安装python
![[Pasted image 20260131111137.png]]
这个是捆绑安装的，安装一个python就会一起安装另外三个

可以在命令行或者终端输入`python --version`，查看版本

## python简单语法
python相对于c,注重于格式规范（PEP原则，[PEP 8：Python 代码风格指南 --- PEP 8: The Style Guide for Python Code](https://pep8.org/)），强调代码可读性和一致性


### 一、 基础架构：缩进与变量

#### 1. 缩进 (Indentation)

在 C/Java 中，花括号 `{}` 决定作用域；在 Python 中，**缩进**决定作用域。

- **规则**：标准是 4 个空格（不建议用 Tab，严禁混用）。
    
- **本质**：强制你写出格式整齐的代码。
    

#### 2. 变量与引用 (Variables as References)

这是 Python 与 C 最大的不同。

- **C 语言**：`int a = 1` 是找个盒子命名为 a，往里放个 1。
    
- **Python**：`a = 1` 是先在内存里生成一个对象 `1`，然后给它贴个标签叫 `a`。



---

### 二、 数据结构

Python 强大的原因在于它内置了极高抽象的数据结构。

#### 1. 列表 (List) `[]`

动态数组，支持异构数据。

- **切片 (Slicing)**`list[start:end:step]`
    
    
    
    ```Python
    arr = [0, 1, 2, 3, 4, 5]
    print(arr[1:4])   # [1, 2, 3] (左闭右开)
    print(arr[::-1])  # [5, 4, 3, 2, 1, 0] (倒序，非常常用)
    ```
    

#### 2. 字典 (Dictionary) `{}`

基于哈希表 (Hash Map) 实现，查找速度 $O(1)$。

- **用法**：`d = {"key": "value"}`
    
- **技巧**：`d.get("key", default)` 防止 Key不存在报错。
    

#### 3. 元组 (Tuple) `()`

**不可变**的列表。

- **为什么需要它？** 因为列表是可变的，不可哈希 (Unhashable)，不能作为字典的 Key。元组不可变，所以可以作为 Key。
    
- **场景**：函数返回多个值时，本质上返回的是一个元组。
    

#### 4. 集合 (Set) `{}`

无序、不重复的元素集。

- **运算**：天生支持数学集合运算。
    
    
    
    ```Python
    a = {1, 2, 3}
    b = {2, 3, 4}
    print(a & b)  # 交集 {2, 3}
    print(a - b)  # 差集 {1}
    ```
    

---

### 三、 控制流：不仅仅是 If-Else

#### 1. 循环中的 Else (For...Else)

这是 Python 特有的语法。`else` 块会在**循环正常结束**（没有被 `break` 打断）时执行。



```Python
for i in range(5):
    if i == 10:
        break
else:
    print("没有找到 10，循环完整结束")  # 这行会被执行
```

#### 2. 模式匹配 (Match...Case)

Python 3.10 引入，类似 Switch，但更强大（支持解包）。



```Python
data = {"type": "click", "pos": (100, 200)}
match data:
    case {"type": "click", "pos": (x, y)}:
        print(f"Clicked at {x}, {y}")
    case _:
        print("Unknown")
```

---

### 四、 函数

在 Python 中，函数也是对象，可以赋值给变量，也可以作为参数传递。

### 1. 动态参数 (`*args`, `**kwargs`)

- `*args`：接收任意数量的位置参数（打包成元组）。
    
- `**kwargs`：接收任意数量的关键字参数（打包成字典）。
    

```Python
def func(*args, **kwargs):
    print(args)    # (1, 2)
    print(kwargs)  # {'a': 3}

func(1, 2, a=3)
```

### 2. 装饰器 (Decorators) `@`

这是 Python 框架（Flask, Django）的核心魔法。本质是**高阶函数**（Higher Order Function）。

- **作用**：在不修改原函数代码的情况下，给函数增加功能（如日志、耗时统计、权限校验）。
    



```Python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}...")
        return func(*args, **kwargs)
    return wrapper

@logger
def add(x, y):
    return x + y

# 等价于：add = logger(add)
add(1, 2)
```

### 3. Lambda 表达式

匿名函数，通常用于简单的回调。



```Python
pairs = [(1, 'one'), (2, 'two')]
pairs.sort(key=lambda pair: pair[1]) # 按第二个元素排序
```

---

### 五、 面向对象：魔术方法 (Magic Methods)

Python 的 OOP 不像 Java 那样死板。它通过“魔术方法”（双下划线方法，Dunder Methods）来实现**运算符重载**和**鸭子类型**。

### 1. `__init__` 与 `self`

- `__init__` 是构造函数。
    
- `self` 必须显式作为第一个参数，代表实例本身（类似 Java 的 `this`，但必须写出来）。
    

### 2. 核心魔术方法

如果你想让你自定义的对象支持 `len()`、`+`、`print()`，你只需要实现对应的方法：



```Python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # 决定 print(v) 显示什么
    def __str__(self):
        return f"Vector({self.x}, {self.y})"
    
    # 决定 v1 + v2 的行为
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)  # 输出 Vector(4, 6)
```

---

### 六、 Pythonic 语法糖：效率与优雅

### 1. 列表推导式 (List Comprehension)

用一行代码生成列表，比 `for` 循环更快（底层 C 实现）。



```Python
# 新手
res = []
for i in range(10):
    if i % 2 == 0:
        res.append(i * i)

# 高手
res = [i * i for i in range(10) if i % 2 == 0]
```

### 2. 生成器 (Generator) 与 `yield`

处理大数据时神器。它不会一次性把数据生成到内存，而是**用一个生成一个**（Lazy Evaluation）。



```Python
def my_range(n):
    i = 0
    while i < n:
        yield i  # 暂停并返回 i，下次从这里继续
        i += 1

# 不占内存，哪怕 n 是 100 亿
for num in my_range(10000000000):
    pass
```

### 3. 上下文管理器 (`with`)

自动管理资源（如文件、锁、网络连接）的打开和关闭。


```Python

# 不用担心忘记 f.close()
with open("file.txt", "r") as f:
    content = f.read()
```

---

### 七、 类型提示 (Type Hinting)

让我们把 **Python** 和 **Rust** 放在一起看：

从 Python 3.5 开始，为了解决动态语言大型项目维护难的问题，引入了类型提示。

- 注意：这只是**提示**，解释器不会强制校验，但 IDE 会给你报错。
**Python :**

```Python
# 变量名: 类型 = 值
age: int = 18
name: str = "Alice"
active: bool = True


def greeting(name: str) -> str:
    return "Hello " + name

```

**Rust:**

```Rust
// let 变量名: 类型 = 值;
let age: i32 = 18;
let name: &str = "Alice"; // 或者 String
let active: bool = true;
```


---
以上就是python的环境配置和语法环节