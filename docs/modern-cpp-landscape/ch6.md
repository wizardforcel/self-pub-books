# 第 6 章 模块、契约与错误处理模型

前面的章节主要讲“怎么写一段正确的代码”，本章开始转向“如何把代码组织成可维护的大型项目”，以及“程序出错时该怎么处理”。

本章包含四个主题：

1. **模块（C++20）**：替代传统头文件的代码组织方式，解决编译慢、宏泄漏等问题；
2. **错误处理**：异常、`std::error_code`、`std::expected`（C++23），三种错误处理方式的适用边界；
3. **契约（C++26 前瞻）**：`pre`、`post`、`contract_assert`，把前置条件、后置条件直接写在函数声明上；
4. **`std::source_location`（C++20）**：不使用宏也能获取文件名、行号、函数名，用于日志和诊断。

---

## 6.1 模块：接口单元、实现单元与 `import`

### 6.1.1 头文件模型的问题

第 1 章已经讲过，传统的 C++ 代码组织靠 `#include` 头文件。`#include` 的本质是**文本复制**：预处理器把头文件的内容原样粘贴到包含它的每个 `.cpp` 文件里。这带来几个问题：

- **编译慢**：每个源文件都要重新解析一遍头文件，头文件被反复展开。
- **宏泄漏**：头文件里定义的宏会影响所有包含它的源文件，命名冲突难以避免。
- **包含顺序敏感**：有时候换一下 `#include` 的顺序，编译就通不了。
- **接口不清**：编译器很难判断“这个头文件到底导出了什么名字”。

C++20 引入了**模块（modules）**，试图从语言层面解决这些问题。模块不是对头文件的简单包装，而是一种全新的编译单元。

### 6.1.2 模块的基本形态

模块通常分成两个文件：**模块接口单元**（`.cppm`，提供对外接口）和**模块实现单元**（`.cpp`，提供具体实现）。使用者通过 `import` 导入模块，而不是 `#include` 头文件。

下面是一个最小的模块示例：

```cpp
// math.cppm —— 模块接口单元
export module math;

export int add(int a, int b) {
    return a + b;
}

export double multiply(double a, double b) {
    return a * b;
}
```

```cpp
// main.cpp —— 导入模块并使用
import math;

#include <iostream>

int main() {
    std::cout << add(2, 3) << '\n';
    std::cout << multiply(2.5, 4.0) << '\n';
}
```

关键语法说明：

- `export module math;`：声明当前文件是一个名为 `math` 的模块接口单元。
- `export`：把后面的函数、类、变量“导出”。只有导出的内容，别的文件 `import` 之后才能看到。没写 `export` 的内容是模块内部私有的。
- `import math;`：导入模块。它不像 `#include` 那样做文本复制，而是让编译器去读模块编译好的接口信息。
- 在模块文件里，`#include <iostream>` 可以放在 `import` 之后（普通头文件仍然可以继续用）。

### 6.1.3 接口单元 vs 实现单元

大型模块通常把接口和实现分开：

```cpp
// math.cppm —— 接口单元，只声明，不定义实现
export module math;

export int add(int a, int b);
export double multiply(double a, double b);
```

```cpp
// math.cpp —— 实现单元
module math;   // 注意：这里没有 export

int add(int a, int b) {
    return a + b;
}

double multiply(double a, double b) {
    return a * b;
}
```

实现单元用 `module math;`（不加 `export`）表明自己属于 `math` 模块，提供具体的函数定义。编译器和链接器会把接口单元和实现单元结合起来。

### 6.1.4 模块 vs 头文件对比

| 对比项 | 头文件 `#include` | 模块 `import` |
|---|---|---|
| 本质 | 文本复制 | 编译器层面的导入 |
| 编译速度 | 每个翻译单元重复解析 | 模块只编译一次，可复用 |
| 宏泄漏 | 会泄漏 | 默认不泄漏 |
| 包含顺序 | 可能敏感 | 不敏感 |
| 接口可见性 | 难以控制 | `export` 明确控制 |
| 成熟度 | 几十年，工具链完善 | 仍在普及，编译器/构建系统支持参差不齐 |

模块的优势很明显，但现实中要普及还需要时间。因为使用模块，构建系统（CMake 等）也需要支持模块的依赖扫描，目前各工具链的支持程度不一。本书把模块作为概念介绍，第 7 章和第 29 章会再提 CMake 对模块的支持；日常示例仍以头文件为主。

C++23 还引入了 `import std;`，可以一次性导入整个标准库，替代一堆 `#include <xxx>`。但目前各编译器对 `import std` 的支持也在逐步完善中。

> **小结**：模块用 `export module` 声明接口、`import` 导入，避免头文件的文本复制和宏泄漏问题；接口单元和实现单元可分开；模块是 C++20 起的新特性，工具链支持仍在完善中。

---

## 6.2 异常、`std::error_code` 与 `std::expected`

程序运行时出错是常态：文件不存在、网络超时、输入非法、内存不足…… C++ 提供多种错误处理方式，没有一种是绝对的银弹，关键是根据场景选型。

### 6.2.1 异常：传统的错误传播机制

C++ 异常使用 `throw` 抛出，`try / catch` 捕获。异常适合“异常情况”（即不应该经常发生、发生后需要中断当前流程并向上层传播的错误）。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <stdexcept>

std::string read_file(const std::string& path) {
    std::ifstream in(path);
    if (!in) {
        throw std::runtime_error("无法打开文件: " + path);
    }
    std::string content((std::istreambuf_iterator<char>(in)),
                        std::istreambuf_iterator<char>());
    return content;
}

int main() {
    try {
        std::string text = read_file("不存在的文件.txt");
        std::cout << text << '\n';
    } catch (const std::exception& e) {
        std::cerr << "捕获到异常：" << e.what() << '\n';
    }
}
```

异常的优点是：错误信息可以自动沿调用栈向上传播，中间层不需要显式处理；缺点是：控制流变得不明显（读代码时不知道哪里可能抛异常），而且抛异常有运行期开销。

### 6.2.2 `std::error_code`：系统错误码（C++11）

对于“可预见的、不算异常的错误”，比如打开文件失败、网络超时，C++11 提供了 `std::error_code`。它表示一个系统或自定义错误码，不抛异常，通过返回值传递。

```cpp
#include <iostream>
#include <fstream>
#include <system_error>
#include <string>

std::error_code read_file(const std::string& path, std::string& out) {
    std::ifstream in(path);
    if (!in) {
        return std::make_error_code(std::errc::no_such_file_or_directory);
    }
    out.assign((std::istreambuf_iterator<char>(in)),
               std::istreambuf_iterator<char>());
    return {};   // 空的 error_code 表示成功
}

int main() {
    std::string content;
    std::error_code ec = read_file("不存在的文件.txt", content);
    if (ec) {
        std::cerr << "错误：" << ec.message() << '\n';
    } else {
        std::cout << content << '\n';
    }
}
```

`std::error_code` 的优点是没有异常的控制流干扰，适合高性能或禁止异常的环境；缺点是每次调用都要检查返回值，容易漏检。

### 6.2.3 `std::expected`：值或错误的类型安全容器（C++23）

C++23 引入了 **`std::expected<T, E>`**，表示“要么是一个成功的值 `T`，要么是一个错误 `E`”。它把“返回值或错误”封装成一个类型，调用者必须显式处理错误才能取出值，比 `std::error_code` 更安全，比异常更轻量。

```cpp
#include <iostream>
#include <expected>
#include <string>
#include <system_error>

std::expected<std::string, std::error_code> read_file(const std::string& path) {
    std::ifstream in(path);
    if (!in) {
        return std::unexpected(std::make_error_code(std::errc::no_such_file_or_directory));
    }
    std::string content((std::istreambuf_iterator<char>(in)),
                        std::istreambuf_iterator<char>());
    return content;
}

int main() {
    auto result = read_file("不存在的文件.txt");
    if (result) {
        std::cout << *result << '\n';           // 成功，取出值
    } else {
        std::cerr << "错误：" << result.error().message() << '\n';  // 取出错误
    }
}
```

`std::expected` 的关键特性：

- `result.operator bool()`：判断是否有值（成功）。
- `*result`：取出值，如果没有值会抛异常。
- `result.error()`：取出错误对象。
- C++23 还提供了单子式操作（如 `and_then`、`or_else`、`transform`），可以把多个可能失败的操作链式组合，避免层层嵌套的 `if`。

### 6.2.4 三种方式怎么选

| 方式 | 适用场景 | 优点 | 缺点 |
|---|---|---|---|
| 异常（`throw` / `catch`） | 真正异常、罕见但严重的错误 | 控制流自动向上传播，中间层无需处理 | 控制流不明显，有运行期开销 |
| `std::error_code` | 可预见的系统/IO 错误、禁止异常的环境 | 无异常开销，适合高性能场景 | 容易漏检返回值 |
| `std::expected`（C++23） | 函数返回“值或错误”的场景 | 类型安全，强制处理错误，可链式组合 | C++23 才有，旧编译器不可用 |

选型口诀：**真异常用异常；系统错误码用 `error_code`；函数返回值或错用 `expected`。** 同一个项目里尽量统一一种风格，不要三种混用。

> **小结**：C++ 提供异常、`std::error_code`、`std::expected` 三种错误处理机制，分别对应“异常情况”“系统错误”“值或错误返回”。`std::expected` 是 C++23 的新设施，兼具类型安全和轻量。

---

## 6.3 契约（C++26 前瞻）：`pre`、`post`、`contract_assert`

### 6.3.1 什么是契约

**契约（contract）** 是指对函数的前置条件（precondition）、后置条件（postcondition）和断言（assertion）的正式说明。简单说，就是告诉调用者和编译器：

- 调用这个函数之前，参数必须满足什么条件（`pre`）；
- 函数返回后，结果和状态必须满足什么条件（`post`）；
- 函数内部某个点必须为真（`contract_assert`）。

契约不是新的错误处理机制，而是一种**接口文档 + 编译期/运行期检查**的结合。

C++26 正在标准化契约语法（基于 P2900 等提案）。当前处于前瞻阶段，各编译器支持情况不一，生产环境请谨慎。

### 6.3.2 契约语法示例

下面的语法基于 C++26 契约提案（前瞻）：

```cpp
// 除法函数：除数不能为零，结果也是有限的
double divide(double a, double b)
    pre (b != 0)
    post (r : std::isfinite(r))
{
    return a / b;
}
```

- `pre (b != 0)`：前置条件，调用者保证传入的 `b` 不为零。
- `post (r : std::isfinite(r))`：后置条件，`r` 是返回值，保证返回结果是有限数。
- 如果前置条件不满足，属于调用方 bug；如果后置条件不满足，属于被调方 bug。

还可以用 `contract_assert` 在函数内部写断言：

```cpp
void process(int index)
    pre (index >= 0)
{
    contract_assert(index < 100);  // 内部断言：index 必须小于 100
    // ...
}
```

### 6.3.3 契约的检查级别

契约可以在不同的**构建模式**下表现不同：

| 构建模式 | 契约行为 |
|---|---|
| 释放（release） | 契约不检查，零开销 |
| 调试（debug） | 违反契约时终止程序或抛异常 |
| 测试（test） | 违反契约时记录但不终止，用于测试覆盖 |

这意味着契约主要用于开发和测试阶段捕捉程序员错误，发布版本可以彻底关闭，没有运行期开销。

### 6.3.4 与 `assert` 的区别

C 语言风格的 `assert` 是运行期宏，只能写在函数体内。契约的 `pre` / `post` 写在函数**声明**上，是接口的一部分，编译器和静态分析工具可以直接读取；`contract_assert` 则更接近 `assert`，但集成在契约系统里，可以统一开关。

> **小结**：契约（C++26 前瞻）用 `pre`、`post`、`contract_assert` 声明前置条件、后置条件和内部断言；支持按构建模式开关，发布版本零开销。当前支持情况有限，请标注“前瞻”，不要在生产核心代码中依赖。

---

## 6.4 `std::source_location` 与诊断信息

### 6.4.1 为什么不直接用 `__FILE__` 和 `__LINE__`

传统上，如果要在日志里记录“哪个文件的哪一行打印了这条信息”，需要用预处理宏：

```cpp
#define LOG(msg) do { \
    std::clog << __FILE__ << "(" << __LINE__ << "): " << msg << '\n'; \
} while (0)
```

这有几个问题：
- 宏不尊重作用域和命名空间；
- `__FILE__` 和 `__LINE__` 在宏展开点求值，不能传递给另一个函数；
- 无法获取函数名（`__func__` 可以，但语法零散）。

C++20 引入了 `std::source_location`，提供一个类型安全的替代方案。

### 6.4.2 基本用法

```cpp
#include <iostream>
#include <source_location>
#include <string_view>

void log(std::string_view message,
         const std::source_location& location = std::source_location::current())
{
    std::clog << location.file_name() << '('
              << location.line() << ':'
              << location.column() << ") `"
              << location.function_name() << "`: "
              << message << '\n';
}

void foo() {
    log("foo 被调用");
}

int main() {
    log("程序启动");
    foo();
}
```

可能的输出：

```
ch6_source_location.cpp(25:8) `int main()`: 程序启动
ch6_source_location.cpp(20:8) `void foo()`: foo 被调用
```

关键点：

- `std::source_location::current()` 是一个静态工厂函数，它返回**调用处**的源码位置（文件名、行号、列号、函数名）。
- 把它作为默认参数，编译器会在**调用点**而不是函数内部求值，所以 `file_name()` 和 `line()` 真正反映的是调用者的位置。
- 无需宏，就是一个普通的 C++ 类型。

### 6.4.3 成员函数

| 成员 | 含义 |
|---|---|
| `line()` | 行号 |
| `column()` | 列号 |
| `file_name()` | 文件名 |
| `function_name()` | 函数名（如果有） |

### 6.4.4 典型应用

- **日志系统**：自动记录日志产生的位置，不需要手动写文件名和行号；
- **测试框架**：断言失败时报告确切位置；
- **调试工具**：在崩溃或错误处理路径中附加上下文信息。

`std::source_location` 需要 `<source_location>` 头文件，对应的特性测试宏是 `__cpp_lib_source_location`（C++20）。

> **小结**：`std::source_location`（C++20）无需宏就能获取调用处的文件名、行号、列号、函数名，作为默认参数使用最方便，适合日志、测试和诊断场景。

---

## 本章小结

本章我们学习了代码组织与错误处理的核心知识：

1. **模块（C++20）**：`export module` 声明接口单元、`import` 导入模块，解决头文件的编译慢、宏泄漏问题；接口单元和实现单元可分开；`import std`（C++23）可一次性导入标准库。
2. **错误处理三种方式**：异常适合罕见的严重错误；`std::error_code` 适合可预见的系统错误；`std::expected`（C++23）表示“值或错误”，类型安全且可链式组合。
3. **契约（C++26 前瞻）**：`pre`、`post`、`contract_assert` 声明前置条件、后置条件和断言；按构建模式开关，发布版本零开销。
4. **`std::source_location`（C++20）**：不使用宏也能获取文件名、行号、函数名，适合日志和诊断。

下一章是“CMake 与 vcpkg：现代 C++ 工程化基础”，它是全书的工具分水岭：从本章起，示例告别 cxxsh 单文件脚本，进入真实项目的工程化构建。

## 练习

1. 写一个名为 `utils` 的模块接口单元，导出一个 `greet()` 函数，再在另一个文件里 `import utils` 并调用。
2. 用 `std::expected<int, std::string>` 重写一个“除法”函数，除数为零时返回错误信息，否则返回商。
3. 给一个计算阶乘的函数加上契约前置条件（输入必须非负）和后置条件（结果必须大于等于 1），注意标注 C++26 前瞻。
4. 写一个 `log_message` 函数，使用 `std::source_location::current()` 作为默认参数，在 `main` 和另一个函数里分别调用，观察打印的行号是否分别指向两个调用点。
5. 对比同一个错误场景用异常、`std::error_code`、`std::expected` 三种写法的代码差异，写下各自的优缺点。
