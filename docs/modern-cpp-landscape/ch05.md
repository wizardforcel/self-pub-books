# 第 5 章 模板、泛型与编译期抽象

如果类是“对象的图纸”，那模板就是“图纸的图纸”——它不是一个具体的类或函数，而是一份可以生成很多个类或函数的模子。C++ 的模板系统非常强大，从 C++98 的基本函数模板、类模板，到 C++11 的可变参数模板，再到 C++20 的概念（concepts），模板一直是现代 C++ 泛型编程和编译期抽象的核心。

本章我们学习：函数模板、类模板、变量模板；CTAD（类模板参数推导）；可变参数模板与折叠表达式；`if constexpr` 编译期分支；概念与约束；以及现代替代传统模板元编程的 `type_traits`、`integer_sequence` 与 `ratio`。

> **工具约定**：本章及第 7 章之前的单文件示例均可用 `cxxsh` 像脚本一样运行。涉及 C++20/23 专有语法（概念、折叠表达式等）的示例，请确保编译器支持 C++20 及以上，并用 `-std=c++20` 编译。

---

## 5.1 函数模板、类模板与变量模板

### 5.1.1 函数模板：一份代码，适用多种类型

假设我们想写一个返回两个数中较大值的函数。对于 `int` 我们可以写：

```cpp
int max(int a, int b) {
    return a > b ? a : b;
}
```

但如果还要处理 `double`、`std::string`，就得为每种类型都写一遍几乎一模一样的代码。这时候函数模板就派上用场了：

```cpp
template <typename T>
T max(T a, T b) {
    return a > b ? a : b;
}
```

`template <typename T>` 的意思是：“定义一个模板，`T` 是一个待定的类型参数”。调用时，编译器会根据你传入的实参自动推导 `T` 是什么：

```cpp
#include <iostream>
#include <string>

template <typename T>
T max(T a, T b) {
    return a > b ? a : b;
}

int main() {
    std::cout << max(3, 5) << '\n';           // T = int
    std::cout << max(3.5, 2.5) << '\n';       // T = double
    std::cout << max(std::string("apple"), std::string("banana")) << '\n';  // T = std::string
}
```

函数模板本身不是函数，编译器只有在你实际调用（实例化）时，才会根据 `T` 的具体类型生成真正的函数代码。`max(3, 5)` 会生成 `int max(int, int)`，`max(3.5, 2.5)` 会生成 `double max(double, double)`。

函数模板也可以显式指定类型，不让编译器推导：

```cpp
max<double>(3, 5);   // 明确指定 T = double，3 和 5 会转成 double
```

### 5.1.2 类模板：一份类图纸，适用多种类型

类模板的思路和函数模板一样，只是作用于类。标准库里的 `std::vector`、`std::pair`、`std::tuple` 全都是类模板。

我们自己写一个简单的容器模板：

```cpp
#include <iostream>

template <typename T>
class Box {
public:
    Box(T value) : value_(value) {}
    T get() const { return value_; }
    void set(T value) { value_ = value; }

private:
    T value_;
};

int main() {
    Box<int> int_box(42);
    Box<std::string> str_box("hello");
    
    std::cout << int_box.get() << '\n';
    std::cout << str_box.get() << '\n';
}
```

`Box<int>` 和 `Box<std::string>` 是两个不同的类，编译器分别为它们生成代码。类模板的每个成员函数，只有在被实际使用时才会被实例化，这可以避免不必要的代码膨胀。

类模板也可以有多个类型参数：

```cpp
template <typename Key, typename Value>
struct Pair {
    Key key;
    Value value;
};
```

### 5.1.3 变量模板：编译期常量的模板化（C++14）

C++14 引入了**变量模板（variable template）**，允许定义一个随类型参数变化的编译期常量：

```cpp
#include <iostream>

template <typename T>
constexpr T pi = T(3.1415926535897932385);

int main() {
    std::cout << pi<int> << '\n';    // 3
    std::cout << pi<double> << '\n'; // 3.141592653589793
    std::cout << pi<float> << '\n';  // 3.141593
}
```

`pi<int>`、`pi<double>` 是不同的常量，各自具有对应的类型。变量模板常用于定义与类型相关的编译期常数，比如数学常数、类型特征值等。

> **小结**：函数模板和类模板让一份代码适用多种类型；变量模板（C++14）把编译期常量也模板化。模板在调用/使用时才由编译器实例化，这叫“惰性生成”。

---

## 5.2 模板参数推导与 CTAD

### 5.2.1 函数模板的参数推导

调用函数模板时，你通常不需要指定模板参数，编译器会从函数实参推导出来：

```cpp
template <typename T>
void print(T value) {
    std::cout << value << '\n';
}

print(42);          // T 推导为 int
print(3.14);        // T 推导为 double
print("hello");     // T 推导为 const char*
```

推导遵循和 `auto` 类似的规则：通常会丢弃顶层 `const` 和引用，除非你显式写成 `T&` 或 `const T&`。

### 5.2.2 CTAD：类模板参数推导（C++17）

在 C++17 之前，使用类模板必须显式写出所有模板参数，很啰嗦：

```cpp
std::pair<int, std::string> p(1, "one");  // C++17 前必须写 <int, std::string>
```

C++17 引入了 **CTAD（Class Template Argument Deduction，类模板参数推导）**，让编译器从构造函数的实参推导类模板的参数：

```cpp
std::pair p(1, "one");          // 推导为 std::pair<int, const char*>
std::tuple t(1, 2.5, "three");  // 推导为 std::tuple<int, double, const char*>
std::vector v = {1, 2, 3};      // 推导为 std::vector<int>
```

这意味着你不再需要写 `std::vector<int> v = {1, 2, 3}` 里的 `<int>`，编译器会从初始化列表推断出来。

不过 CTAD 不能推导所有情况。如果编译器推导不出唯一类型，或者类没有合适的构造函数，你需要显式指定模板参数，或者提供**推导指引（deduction guide）**。

### 5.2.3 推导指引：教编译器怎么推导

对于自定义的类模板，编译器会根据构造函数自动生成推导候选，但有时候自动推导不符合预期，这时可以手写推导指引：

```cpp
#include <iostream>
#include <string>

template <typename T>
struct Wrapper {
    T value;
    Wrapper(T v) : value(v) {}
};

// 推导指引：Wrapper(int) 推导为 Wrapper<int>
Wrapper(int) -> Wrapper<int>;

int main() {
    Wrapper w(42);   // 推导为 Wrapper<int>
    std::cout << w.value << '\n';
}
```

推导指引的语法是 `构造函数签名 -> 模板实例化结果;`。它告诉编译器：“如果看到这种构造方式，就推导成那个模板类型。”

> **小结**：函数模板参数由实参推导；CTAD（C++17）让类模板也能从构造函数参数推导模板参数；推导指引可用于微调自定义类模板的推导行为。

---

## 5.3 可变参数模板与折叠表达式

### 5.3.1 可变参数模板：参数个数不固定的模板

C++11 引入了**可变参数模板（variadic template）**，允许模板接受任意数量的参数。它用 `typename... Args` 或 `class... Args` 表示一个**参数包（parameter pack）**。

```cpp
#include <iostream>

template <typename... Args>
void print_all(Args... args) {
    // 怎么把 args 里的每个元素都打印出来？
}
```

问题在于，`args` 是一个包，不能直接像普通变量那样遍历。我们需要**递归展开**：

```cpp
#include <iostream>

// 终止条件：没有参数时
void print_all() {
    std::cout << '\n';
}

// 递归步骤：处理第一个，剩下的交给下一次调用
template <typename T, typename... Rest>
void print_all(T first, Rest... rest) {
    std::cout << first << ' ';
    print_all(rest...);   // 把剩下的参数再传进去
}

int main() {
    print_all(1, 2.5, std::string("hello"), 'c');
}
```

输出：

```
1 2.5 hello c 
```

递归展开的逻辑是：`print_all(1, 2.5, "hello", 'c')` 先打印 1，再调用 `print_all(2.5, "hello", 'c')`，依次类推，直到 `print_all()` 匹配到无参版本终止。

### 5.3.2 折叠表达式：一行代码处理参数包（C++17）

递归展开虽然能用，但代码冗长。C++17 引入了**折叠表达式（fold expression）**，可以用类似“折叠”的语法一次性对参数包里的所有元素应用一个运算符：

```cpp
#include <iostream>

template <typename... Args>
auto sum(Args... args) {
    return (args + ...);   // 折叠表达式：把所有参数用 + 连起来
}

int main() {
    std::cout << sum(1, 2, 3, 4, 5) << '\n';   // 15
    std::cout << sum(1.5, 2.5) << '\n';        // 4.0
}
```

`(args + ...)` 叫**一元右折叠**，展开后相当于 `1 + (2 + (3 + (4 + 5)))`。

折叠表达式有四种形式：

| 形式 | 含义 | 展开示例（参数为 a, b, c） |
|---|---|---|
| `(pack op ...)` | 一元右折叠 | `a op (b op c)` |
| `(... op pack)` | 一元左折叠 | `(a op b) op c` |
| `(pack op ... op init)` | 二元右折叠 | `a op (b op (c op init))` |
| `(init op ... op pack)` | 二元左折叠 | `((init op a) op b) op c` |

其中 `op` 可以是 `+`、`*`、`&&`、`||`、`,`、`<` 等大多数二元运算符。

折叠表达式最常见的用法之一是把多个参数都传给一个函数：

```cpp
#include <iostream>

template <typename... Args>
void print_all(Args... args) {
    (std::cout << ... << args) << '\n';  // 折叠表达式，依次输出
}

int main() {
    print_all(1, 2.5, "hello");
}
```

`(std::cout << ... << args)` 展开为 `std::cout << 1 << 2.5 << "hello"`，非常简洁。

> **小结**：可变参数模板接受任意数量参数，传统上用递归展开；C++17 折叠表达式让参数包的处理可以写成一行，代码更简洁、更高效。

---

## 5.4 `if constexpr` 与编译期分支

### 5.4.1 普通 `if` 的局限

在模板里，有时候想根据模板参数的类型做不同的处理。普通 `if` 是运行期分支，两个分支都要能编译通过：

```cpp
template <typename T>
void print_info(T value) {
    if (std::is_integral_v<T>) {      // T 是整数类型
        std::cout << "整数： " << value << '\n';
    } else {
        std::cout << "其他： " << value << '\n';
    }
}
```

这段代码看起来没问题，但如果 `T` 是 `std::string`，`std::cout << value` 没问题；如果 `T` 是某种不支持 `<<` 的类型，编译器在检查 `if` 里的 `std::cout << value` 时就会报错，即使那个分支在运行时永远不会执行。

### 5.4.2 `if constexpr`：编译期条件编译

C++17 引入了 `if constexpr`，它在**编译期**就判断条件，不满足条件的分支会被直接舍弃，不会实例化、不会检查语法错误：

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void print_info(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "整数： " << value << '\n';
    } else {
        std::cout << "其他： " << value << '\n';
    }
}
```

如果 `T` 是 `int`，编译器只保留第一个分支，丢弃第二个；如果 `T` 是 `std::string`，只保留第二个分支。丢弃的分支里的代码即使有错误，也不会导致编译失败（前提是那个分支本身不影响模板实例化）。

`if constexpr` 的条件必须是**编译期常量表达式**（能转成 bool 的常量）。

### 5.4.3 典型应用：消除重复代码

看一个实际例子：把不同类型的值转换成字符串。

```cpp
#include <iostream>
#include <string>
#include <type_traits>
#include <sstream>

template <typename T>
std::string to_string(T value) {
    if constexpr (std::is_same_v<T, std::string>) {
        return value;                       // 已经是字符串，直接返回
    } else if constexpr (std::is_arithmetic_v<T>) {
        return std::to_string(value);       // 数字类型，用标准库转换
    } else {
        std::ostringstream oss;
        oss << value;                       // 其他类型，用流输出
        return oss.str();
    }
}

int main() {
    std::cout << to_string(42) << '\n';           // 整数
    std::cout << to_string(3.14) << '\n';         // 浮点数
    std::cout << to_string(std::string("hello")) << '\n';  // 字符串
}
```

没有 `if constexpr`，`std::to_string(std::string)` 这种调用会编译失败，因为标准库的 `std::to_string` 不接受 `std::string`。有了 `if constexpr`，不匹配的分支被舍弃，同一份代码就能处理多种类型。

> **小结**：`if constexpr`（C++17）在编译期做分支选择，不满足条件的分支被舍弃，不会报语法错误。它让模板代码可以根据类型特征“按需编译”，是替代复杂 SFINAE 的现代写法。

---

## 5.5 概念与约束：从 SFINAE 到 `concepts`

### 5.5.1 为什么需要概念

模板很强大，但有一个很大的痛点：**错误信息难以理解**。如果你调用一个模板函数，传入了不合适的类型，编译器可能会吐出几十行模板实例化的错误堆栈，最后才发现是“类型不满足某个隐含要求”。

比如这个函数要求 `T` 必须能用 `+` 相加：

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}
```

如果你传入两个不支持 `+` 的类型，编译器会报错，但错误信息会指向模板内部的 `a + b`，而不是调用处“类型不满足要求”。而且这个要求（`T` 必须支持 `+`）并没有显式写出来，阅读代码的人很难一眼看出。

C++20 引入了**概念（concepts）**，让你可以显式声明模板参数需要满足什么条件：

```cpp
#include <concepts>

template <typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
};
```

`concept Addable` 定义了一个名为 `Addable` 的约束：类型 `T` 必须支持 `a + b`，且返回类型是 `T`。

然后在模板里使用：

```cpp
template <Addable T>
T add(T a, T b) {
    return a + b;
}
```

现在如果传入不支持 `+` 的类型，编译器会直接报错说“类型不满足 Addable 概念”，错误信息清晰很多，而且意图一目了然。

### 5.5.2 标准库提供的常用概念

标准库 `<concepts>` 头文件提供了一大批开箱即用的概念，覆盖了常见的类型要求：

| 概念 | 含义 |
|---|---|
| `std::same_as<T, U>` | `T` 和 `U` 是同一类型 |
| `std::convertible_to<T, U>` | `T` 能隐式转换为 `U` |
| `std::integral<T>` | `T` 是整数类型 |
| `std::floating_point<T>` | `T` 是浮点类型 |
| `std::invocable<F, Args...>` | `F` 可以用 `Args...` 调用 |
| `std::regular<T>` | `T` 是“常规类型”（可复制、可比较、可析构等） |
| `std::totally_ordered<T>` | `T` 支持全序比较（`<` `<=` `>` `>=`） |

示例：

```cpp
#include <iostream>
#include <concepts>

template <std::integral T>
T gcd(T a, T b) {
    while (b != 0) {
        T temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}

int main() {
    std::cout << gcd(48, 18) << '\n';   // 6
    // std::cout << gcd(3.14, 2.5) << '\n';  // 编译错误：double 不是 integral
}
```

`template <std::integral T>` 是**简写模板（abbreviated function template）**，等价于：

```cpp
template <typename T>
requires std::integral<T>
T gcd(T a, T b) { ... }
```

`requires` 子句是显式写约束的方式，简写版只是语法糖。

### 5.5.3 约束偏序与合取/析取

当多个概念都适用时，编译器需要决定哪个更“特殊”，这叫**约束偏序（constraint subsumption）**。简单说：要求越严格的概念越优先。

你也可以用逻辑运算符组合概念：

- `A && B`：同时满足 A 和 B（合取）
- `A || B`：满足 A 或 B 中的任意一个（析取）
- `!A`：不满足 A（否定）

```cpp
template <typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template <Numeric T>
T square(T x) {
    return x * x;
}
```

这里 `Numeric` 表示“整数或浮点类型”，`square` 只接受数字类型。

> **小结**：概念（C++20）显式声明模板参数的要求，错误信息清晰，意图明确；标准库提供了 `std::integral`、`std::invocable` 等常用概念；可用 `&&`/`||`/`!` 组合概念，并通过 `requires` 子句或简写模板使用。

---

## 5.6 模板元编程的现代替代：`type_traits`、`integer_sequence` 与 `ratio`

### 5.6.1 从 TMP 到类型特征

在 C++11 之前，程序员常用“模板元编程（Template Metaprogramming, TMP）”在编译期做类型判断、计算等工作，代码通常很晦涩：

```cpp
// 老式 TMP 风格（晦涩难读）
template <int N> struct Factorial {
    static constexpr int value = N * Factorial<N-1>::value;
};
template <> struct Factorial<0> { static constexpr int value = 1; };
```

现代 C++ 提供了 `<type_traits>` 头文件，里面包含大量现成的编译期类型查询工具，不再需要手写递归模板结构体。

常用的类型特征：

```cpp
#include <iostream>
#include <type_traits>

int main() {
    std::cout << std::is_integral_v<int> << '\n';       // 1
    std::cout << std::is_integral_v<double> << '\n';    // 0
    std::cout << std::is_pointer_v<int*> << '\n';       // 1
    std::cout << std::is_same_v<int, int> << '\n';      // 1
    std::cout << std::is_same_v<int, long> << '\n';     // 0
    std::cout << std::is_virtual_base_of_v ... << '\n'; // 等等
}
```

这些特征都是编译期常量（`true`/`false`），可以和 `static_assert`、`if constexpr` 配合使用：

```cpp
template <typename T>
void process(T value) {
    static_assert(std::is_integral_v<T>, "process 只接受整数类型");
    // ...
}
```

### 5.6.2 `std::integer_sequence`：编译期整数序列（C++14）

`std::integer_sequence` 是一个编译期整数序列，常配合参数包展开使用，实现“对索引做某事”的模式：

```cpp
#include <iostream>
#include <utility>
#include <tuple>

template <typename Tuple, std::size_t... Is>
void print_tuple_impl(const Tuple& t, std::index_sequence<Is...>) {
    ((std::cout << std::get<Is>(t) << ' '), ...);  // 折叠表达式
    std::cout << '\n';
}

template <typename... Args>
void print_tuple(const std::tuple<Args...>& t) {
    print_tuple_impl(t, std::index_sequence_for<Args...>{});
}

int main() {
    auto t = std::make_tuple(1, 2.5, std::string("hello"));
    print_tuple(t);   // 1 2.5 hello 
}
```

`std::index_sequence_for<Args...>` 会产生 `0, 1, 2, ..., N-1` 序列，然后通过折叠表达式依次访问 tuple 的每个元素。

### 5.6.3 `std::ratio`：编译期有理数运算（C++11）

`std::ratio` 表示一个编译期的有理数（分数），常用于表示时间单位（`std::chrono` 就是基于它构建的）：

```cpp
#include <iostream>
#include <ratio>

int main() {
    using minute = std::ratio<60>;           // 60/1 秒
    using second = std::ratio<1>;            // 1/1 秒
    
    using minute_to_second = std::ratio_multiply<minute, std::ratio<1>>;  // 60
    
    std::cout << minute_to_second::num << '/' << minute_to_second::den << '\n';  // 60/1
}
```

`std::ratio` 支持 `ratio_add`、`ratio_subtract`、`ratio_multiply`、`ratio_divide`、`ratio_less` 等元函数，所有计算都在编译期完成，结果为最简分数。

> **小结**：`<type_traits>` 提供了丰富的编译期类型查询，替代了大量手写 TMP 代码；`std::integer_sequence` 配合参数包展开可以处理编译期索引序列；`std::ratio` 提供编译期有理数运算，广泛用于时间单位等领域。现代 C++ 里，大多数传统 TMP 需求已经有更简洁的标准设施替代。

---

## 本章小结

本章我们学习了模板与编译期抽象的核心知识：

1. **函数模板、类模板、变量模板**：一份代码适用多种类型；模板在实际使用时才由编译器实例化。
2. **CTAD（C++17）**：类模板参数可从构造函数实参推导，推导指引可微调推导行为。
3. **可变参数模板与折叠表达式**：接受任意数量参数；C++17 折叠表达式让参数包处理一行搞定。
4. **`if constexpr`（C++17）**：编译期分支，不满足条件的分支被舍弃，避免 SFINAE 风格的晦涩代码。
5. **概念与约束（C++20）**：显式声明模板参数要求，错误信息清晰；标准库提供 `std::integral`、`std::invocable` 等常用概念。
6. **现代 TMP 替代品**：`std::is_*_v` 类型特征、`std::integer_sequence` 编译期序列、`std::ratio` 编译期有理数，覆盖了大多数传统 TMP 场景。

下一章是“模块、契约与错误处理模型”，学习 C++20 模块、异常与 `std::expected`、C++26 契约前瞻以及 `std::source_location`。

## 练习

1. 写一个函数模板 `sum`，接受任意数量参数并返回它们的总和，要求所有参数类型相同；用折叠表达式实现。
2. 定义一个类模板 `Pair<T, U>`，并尝试 CTAD：`Pair p(1, 3.14);` 应该推导为 `Pair<int, double>`。
3. 写一个概念 `Printable`，要求类型支持 `std::cout << value`，并用它约束一个 `print` 模板函数。
4. 用 `if constexpr` 写一个 `describe` 模板，分别处理整型、浮点型和其他类型，打印不同描述。
5. 用 `std::is_integral_v` 和 `static_assert` 给一个模板函数加上“只接受整数”的编译期检查。
