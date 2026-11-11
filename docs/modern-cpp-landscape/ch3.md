# 第 3 章 函数、Lambda 与可调用对象

如果把程序比作一道菜，变量就是食材，函数就是“把食材加工成菜肴的步骤”。没有函数，所有的逻辑都要堆在 `main` 里，代码很快就会变成一团乱麻。

本章我们学习如何定义函数、如何让函数灵活地接收参数、如何用 Lambda 表达式创建“临时小函数”，以及如何把函数当成对象来传递和存储。这些是后面学习模板、标准库算法、并发编程的基础。

---

## 3.1 函数重载、默认参数与 `constexpr` 函数

### 3.1.1 函数的基本结构

先回顾最简单的函数定义：

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

int main() {
    int result = add(3, 5);
    std::cout << "3 + 5 = " << result << '\n';
}
```

函数由几部分组成：

- 返回类型：`int`，表示函数执行完后返回什么类型的值。如果不返回值，写 `void`。
- 函数名：`add`，用来调用这个函数。
- 参数列表：`(int a, int b)`，函数运行时需要的输入。
- 函数体：`{ return a + b; }`，具体的执行步骤。

调用函数时，会把实参（比如 `3` 和 `5`）复制给形参（`a` 和 `b`），然后执行函数体，最后把结果返回。

### 3.1.2 函数重载：同一个名字，不同的参数

C++ 允许定义多个名字相同但参数不同的函数，这叫**函数重载（function overloading）**：

```cpp
#include <iostream>
#include <string>

int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}

std::string add(const std::string& a, const std::string& b) {
    return a + b;
}

int main() {
    std::cout << add(3, 5) << '\n';            // 调用 int 版本
    std::cout << add(3.5, 2.5) << '\n';        // 调用 double 版本
    std::cout << add(std::string("hello"), std::string(" world")) << '\n';  // 调用 string 版本
}
```

编译器根据你传入的实参类型，自动选择匹配的版本。`add(3, 5)` 是 `int`，所以选第一个；`add(3.5, 2.5)` 是 `double`，选第二个；传入 `std::string`，选第三个。

重载不是“看返回类型”，只看参数列表。下面这段是错误的，因为两个函数参数完全一样，编译器分不清该调哪个：

```cpp
int add(int a, int b);    // 错误：与下面重复
double add(int a, int b); // 错误：只差返回类型，不构成重载
```

### 3.1.3 默认参数：让调用更灵活

有时候函数的某个参数大部分时候都传同一个值，这时可以给它一个**默认参数**。调用时可以不传这个参数，编译器会自动补上默认值：

```cpp
#include <iostream>

void greet(const std::string& name, int times = 1) {
    for (int i = 0; i < times; ++i) {
        std::cout << "你好，" << name << "！\n";
    }
}

int main() {
    greet("张三");              // times 使用默认值 1，打印一次
    greet("李四", 3);           // 明确传 3，打印三次
}
```

默认参数要写在**参数列表的右边**。一旦一个参数有了默认值，它右边的所有参数都必须有默认值：

```cpp
void f(int a, int b = 10, int c = 20);  // 正确
void f(int a = 1, int b, int c = 20);   // 错误：b 没有默认值，却在有默认值的 a 右边
```

默认参数通常写在**函数声明**里（头文件中），而不是定义里，避免重复。

### 3.1.4 `constexpr` 函数：编译期求值的函数

第 2 章提到过 `constexpr` 变量。`constexpr` 也可以修饰函数，表示这个函数**可以在编译期求值**。如果传入的参数都是编译期常量，编译器可以直接在编译时算出结果，生成的代码里直接是常数，运行期完全不需要再计算。

```cpp
#include <iostream>

constexpr int square(int x) {
    return x * x;
}

int main() {
    constexpr int r1 = square(5);   // 编译期算出 25，存入常量
    int x = 6;
    int r2 = square(x);             // 运行期调用，x 不是常量，正常计算
    
    std::cout << r1 << ' ' << r2 << '\n';
}
```

`constexpr` 函数有几条重要规则：

- 函数体里的代码要尽量简单。C++14 以后已经可以写循环、局部变量，但函数必须**只有一条返回语句的路径**（或者说所有路径都要能在编译期求值）。
- 如果 `constexpr` 函数的参数不是常量，它也可以在运行期正常调用，只是没法享受编译期求值的好处。

```cpp
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}

constexpr int f5 = factorial(5);    // 编译期算出 120
```

`constexpr` 函数特别适合用于定义编译期常量、数组大小、模板参数等场景，能把计算从运行期前移到编译期，零运行期开销。

C++23 进一步放宽了 `constexpr` 函数的限制，允许在 `constexpr` 函数里使用 `goto` 和标签，以及更多运行期特性，但这些属于进阶内容，初学者只需要知道“`constexpr` 函数能在编译期算出结果”就够了。

> **小结**：函数重载让同一个名字可以处理不同类型的参数；默认参数让常用参数可以省略；`constexpr` 函数能在编译期求值，减少运行期开销。

---

## 3.2 Lambda 表达式：从 C++11 到 C++23 的完整形态

### 3.2.1 什么是 Lambda

Lambda（λ）表达式是 C++11 引入的“匿名函数”语法。它允许你在需要一个小函数的地方，直接当场写出来，而不必去另外定义一个有名字的函数。

最简单的 Lambda：

```cpp
[] {
    std::cout << "Hello from lambda\n";
}
```

这个表达式本身就是一个“函数对象”，你可以把它存到变量里，稍后调用：

```cpp
auto hello = [] {
    std::cout << "Hello from lambda\n";
};
hello();   // 调用
```

Lambda 的完整语法结构大致是这样：

```cpp
[captures](parameters) -> return_type {
    body
}
```

各部分的意思：

- `[captures]`：捕获列表，说明 Lambda 想要从外部“借用”哪些变量。
- `(parameters)`：参数列表，和普通函数一样。
- `-> return_type`：后置返回类型，可选。如果不写，编译器会自动推导。
- `{ body }`：函数体。

### 3.2.2 捕获列表：Lambda 如何“借用”外部变量

Lambda 定义在某个作用域里，默认情况下它只能使用自己的参数和函数体内定义的变量。如果想用外面的变量，就要在捕获列表里说明。

捕获方式有几种：

| 捕获写法 | 含义 |
|---|---|
| `[]` | 不捕获任何外部变量 |
| `[a]` | 以值捕获变量 `a`（拷贝一份进 Lambda 内部） |
| `[&a]` | 以引用捕获变量 `a`（内部直接引用外部 `a`） |
| `[&]` | 以引用捕获所有用到的外部变量 |
| `[=]` | 以值捕获所有用到的外部变量 |
| `[this]` | 捕获当前对象指针（C++20 起也可写 `[*this]` 捕获当前对象的副本） |

看一个具体的例子：

```cpp
#include <iostream>

int main() {
    int x = 10;
    int y = 20;

    // 值捕获 x，引用捕获 y
    auto f = [x, &y] {
        std::cout << "x = " << x << '\n';  // x 是副本，修改不影响外面
        y = 999;                            // y 是引用，修改会影响外面
        std::cout << "y = " << y << '\n';
    };

    f();
    std::cout << "外面 y = " << y << '\n';  // 999，被 Lambda 修改过了
}
```

输出：

```
x = 10
y = 999
外面 y = 999
```

注意值捕获的陷阱：`[x]` 捕获的是 `x` 在 Lambda 创建时刻的**副本**。如果之后外面的 `x` 变了，Lambda 里的 `x` 不会变：

```cpp
int x = 10;
auto f = [x] { std::cout << x << '\n'; };
x = 20;
f();   // 仍然打印 10
```

### 3.2.3 `mutable`：允许修改值捕获的变量

默认情况下，Lambda 的函数调用操作符是 `const` 的，也就是说，**值捕获的变量在 Lambda 内部不能修改**。如果想修改，需要加上 `mutable` 关键字：

```cpp
#include <iostream>

int main() {
    int count = 0;
    auto f = [count]() mutable {
        ++count;              // 修改的是副本
        std::cout << count << '\n';
    };

    f();   // 1
    f();   // 2
    f();   // 3
    std::cout << "外面 count = " << count << '\n';  // 0，外面的 count 没变
}
```

如果不加 `mutable`，`++count;` 就会编译错误，因为 `count` 是以值捕获的，默认是常量。

### 3.2.4 泛型 Lambda：参数用 `auto`

C++14 起，Lambda 的参数可以用 `auto`，这样 Lambda 就变成了**泛型 Lambda**，可以接受不同类型的参数：

```cpp
#include <iostream>

int main() {
    auto print = [](auto value) {
        std::cout << value << '\n';
    };

    print(42);           // int
    print(3.14);         // double
    print(std::string("hello"));  // std::string
}
```

编译器会为每种不同的参数类型生成一份对应的代码，这和函数模板的效果类似，但语法更轻量。

### 3.2.5 初始化捕获：C++14 的新增

C++14 允许在捕获列表里直接初始化新变量，这叫**初始化捕获（init-capture）**：

```cpp
#include <iostream>

int main() {
    int x = 10;
    auto f = [twice = x * 2] {
        std::cout << twice << '\n';  // 20
    };
    f();
}
```

这里 `twice = x * 2` 在 Lambda 内部创建了一个名为 `twice` 的变量，初始值是 `x * 2`。初始化捕获常用于把一个对象“移动”进 Lambda（比如配合 `std::move`）：

```cpp
auto f = [ptr = std::move(some_unique_ptr)] {
    // 使用 ptr
};
```

### 3.2.6 模板 Lambda：C++20 的显式模板参数

C++20 起，Lambda 可以直接写模板参数列表：

```cpp
#include <iostream>
#include <vector>

int main() {
    auto print_all = []<typename T>(const std::vector<T>& v) {
        for (const auto& x : v) {
            std::cout << x << ' ';
        }
        std::cout << '\n';
    };

    print_all(std::vector<int>{1, 2, 3});
    print_all(std::vector<std::string>{"a", "b", "c"});
}
```

写法是 `[]<typename T>(...) { ... }`，尖括号里是模板参数，和普通模板函数类似，但不用另外定义模板函数。

### 3.2.7 C++23 的小改进

C++23 对 Lambda 还有两个小但实用的改进：

1. **可以省略参数列表的 `()`**。如果 Lambda 没有参数，可以直接写 `[] { ... }`，不需要写 `[]() { ... }`。其实 C++11 起就可以省略，C++23 把这条规则写得更明确。
2. **可以显式对象参数（explicit object parameter）**，写成 `[](this Type& self) { ... }`，用于在 Lambda 里访问当前对象，这比较高级，后面章节遇到再细说。

下面是一个综合示例，展示 Lambda 被当作参数传给另一个函数（这也是标准库算法最常见的用法）：

```cpp
// lambda_demo.cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {5, 2, 8, 1, 9, 3};

    // 用 Lambda 作为排序规则：降序排列
    std::sort(numbers.begin(), numbers.end(),
              [](int a, int b) {
                  return a > b;
              });

    for (int x : numbers) {
        std::cout << x << ' ';
    }
    std::cout << '\n';
}
```

输出：

```
9 8 5 3 2 1
```

这里 Lambda `[](int a, int b) { return a > b; }` 被传给 `std::sort`，作为比较规则。这就是 Lambda 最经典的使用场景：写一个短小的、只在本地使用的函数，不必专门去起名字。

> **小结**：Lambda 是匿名函数对象，语法是 `[捕获](参数) -> 返回类型 { 体 }`；捕获可以是值捕获、引用捕获、混合捕获，也可以捕获所有；`mutable` 允许修改值捕获的副本；`auto` 参数做成泛型 Lambda；初始化捕获可以创建新成员；Lambda 最常用来作为简短回调传给算法。

---

## 3.3 `std::function`、`std::move_only_function` 与 `std::bind_front/back`

### 3.3.1 什么是“可调用对象”

在 C++ 里，能被 `()` 调用的东西不止普通函数，还包括：

- 函数指针；
- Lambda 表达式产生的闭包对象；
- 重载了 `operator()` 的类对象（叫仿函数/函数对象）；
- 成员函数指针。

把这些统统叫做**可调用对象（callable object）**。

### 3.3.2 `std::function`：类型擦除的可调用包装器

有时候我们想把不同的可调用对象统一存起来，或者作为函数参数传递，但又不想关心它到底是函数指针、Lambda 还是仿函数。`std::function` 就是为此设计的，它属于**类型擦除（type erasure）**机制：它可以包装任何可调用对象，对外呈现统一的签名。

```cpp
#include <iostream>
#include <functional>
#include <string>

void print_info(const std::string& msg) {
    std::cout << "函数: " << msg << '\n';
}

int main() {
    // std::function<void(const std::string&)> 表示：接收一个字符串，无返回值
    std::function<void(const std::string&)> f;

    f = print_info;          // 普通函数
    f("hello");

    f = [](const std::string& msg) {
        std::cout << "Lambda: " << msg << '\n';
    };
    f("world");

    // 一个仿函数
    struct Printer {
        void operator()(const std::string& msg) const {
            std::cout << "仿函数: " << msg << '\n';
        }
    };
    f = Printer{};
    f("c++");
}
```

输出：

```
函数: hello
Lambda: world
仿函数: c++
```

`std::function` 的好处是统一接口，但它也有代价：

1. **运行期开销**：`std::function` 通常涉及一次间接调用，可能还有内存分配（如果可调用对象比较大，无法放入小对象缓冲区）。
2. **可为空**：`std::function` 可以是空的，调用空的 `std::function` 会抛出 `std::bad_function_call` 异常，所以调用前最好判断 `if (f)`。

### 3.3.3 `std::move_only_function`：仅移动的可调用包装器（C++23）

`std::function` 要求包装的对象必须**可复制**，因为 `std::function` 本身可以复制。但有些可调用对象只能移动不能复制，比如包含 `std::unique_ptr` 的 Lambda。这种情况下就不能用 `std::function`。

C++23 引入了 `std::move_only_function`，专门处理“只能移动、不能复制”的可调用对象：

```cpp
#include <iostream>
#include <functional>
#include <memory>

int main() {
    auto ptr = std::make_unique<int>(42);

    // 这个 Lambda 捕获了 unique_ptr，只能移动，不能复制
    auto f = std::move_only_function<void()>([ptr = std::move(ptr)]() mutable {
        std::cout << *ptr << '\n';
    });

    f();              // 42
    // auto f2 = f;   // 编译错误：move_only_function 不能复制
    auto f2 = std::move(f);  // 可以移动
    f2();
}
```

`std::move_only_function` 和 `std::function` 用法几乎一样，区别在于它不支持复制，只支持移动。这使得它更适合异步回调、线程任务等场景：任务对象只会被移动，不会被意外复制。

### 3.3.4 `std::bind_front` 与 `std::bind_back`：绑定参数

有时候我们有一个多参数函数，但想固定其中几个参数，产生一个参数更少的新可调用对象。C++20 引入了 `std::bind_front`，C++23 引入了 `std::bind_back`：

```cpp
#include <iostream>
#include <functional>

void print_prefix(int level, const std::string& msg) {
    std::cout << "[" << level << "] " << msg << '\n';
}

int main() {
    // 固定第一个参数为 1，得到一个只需要 msg 的新函数
    auto log_info = std::bind_front(print_prefix, 1);
    log_info("系统启动");   // 等价于 print_prefix(1, "系统启动")

    // C++23：固定最后一个参数
    auto print_hello = std::bind_back(print_prefix, std::string("固定消息"));
    print_hello(3);         // 等价于 print_prefix(3, "固定消息")
}
```

`std::bind_front` 从前往后绑定参数，`std::bind_back` 从后往前绑定。绑定后得到的对象可以放进 `std::function` 或 `std::move_only_function` 里，也可以直接调用。

> **小结**：`std::function` 可以包装任意可调用对象，统一接口，但有运行期开销且要求可复制；`std::move_only_function`（C++23）专为只能移动的可调用对象设计；`std::bind_front/back` 可以预先绑定部分参数，产生新的可调用对象。

---

## 3.4 完美转发与引用折叠

### 3.4.1 转发引用是什么

第 2 章提到过 `T&&` 在模板语境下的特殊性。当函数模板的参数写成 `T&&` 且 `T` 由实参推导时，这个 `T&&` 叫**转发引用（forwarding reference，也叫通用引用）**。它可以绑定左值，也可以绑定右值：

```cpp
template <typename T>
void func(T&& param) {
    // param 可能是左值引用，也可能是右值引用
}
```

- 传入左值 `int x; func(x);` 时，`T` 推导为 `int&`，`T&&` 经过引用折叠变成 `int&`；
- 传入右值 `func(10);` 时，`T` 推导为 `int`，`T&&` 就是 `int&&`。

### 3.4.2 引用折叠规则

当类型推导产生“引用的引用”时，编译器会按照下面的规则把它们“折叠”成一个引用：

| 原始组合 | 折叠结果 |
|---|---|
| `T& &` | `T&` |
| `T& &&` | `T&` |
| `T&& &` | `T&` |
| `T&& &&` | `T&&` |

口诀：**只要有一边是左值引用 `&`，结果就是左值引用；只有两边都是 `&&`，结果才是 `&&`。**

这是转发引用能“左右通吃”的基础。

### 3.4.3 为什么需要 `std::forward`

在模板函数内部，即使参数类型是 `T&&`，这个参数**本身是一个左值**（因为它有名字）。如果你直接把它传下去，接收方永远会收到一个左值，丢失了“原来是右值”的信息：

```cpp
template <typename T>
void wrapper(T&& value) {
    another_func(value);   // value 是左值，永远调用 another_func 的左值版本
}
```

为了保持“左值进左值出、右值进右值出”，需要用 `std::forward<T>(value)` 把原来的值类型“转发”下去：

```cpp
template <typename T>
void wrapper(T&& value) {
    another_func(std::forward<T>(value));
}
```

`std::forward` 的行为是：如果 `T` 是 `int&`，`forward` 结果是 `int&`；如果 `T` 是 `int`，`forward` 结果是 `int&&`。它本身不做任何资源移动，只是恢复类型信息，让接收方能根据原值类别选择合适的重载。

### 3.4.4 完美转发的完整示例

下面这个例子展示了一个简单的工厂函数，把参数完美转发给构造函数：

```cpp
#include <iostream>
#include <string>
#include <utility>

struct Person {
    std::string name;
    int age;

    Person(std::string n, int a) : name(std::move(n)), age(a) {
        std::cout << "Person 构造\n";
    }
};

template <typename... Args>
Person make_person(Args&&... args) {
    return Person(std::forward<Args>(args)...);
}

int main() {
    std::string name = "张三";

    // 左值：name 会被复制
    auto p1 = make_person(name, 25);

    // 右值：临时字符串会被移动
    auto p2 = make_person(std::string("李四"), 30);
}
```

这里 `Args&&...` 是**参数包**（第 5 章会详细讲），`std::forward<Args>(args)...` 把每个参数按原样转发给 `Person` 的构造函数。如果传入左值，构造函数里用复制构造；如果传入右值，用移动构造。这就是“完美转发”：不丢失任何值类别信息，不产生不必要的复制。

### 3.4.5 `std::move` vs `std::forward`

这两个容易混淆，一句话区分：

- `std::move(x)`：**无条件**把 `x` 转成右值引用，表示“我不要这个了，你可以偷它的资源”。
- `std::forward<T>(x)`：**有条件**地保持原类型。如果 `T` 推导出来是左值引用，就转发为左值；如果 `T` 是非引用类型，就转发为右值。

> **小结**：转发引用 `T&&` 能同时绑定左值和右值；引用折叠规则决定了“引用的引用”最终成什么；`std::forward` 把参数原样转发下去，保留值类别；完美转发常用于泛型工厂函数和 `emplace` 风格的接口，避免不必要的复制。

---

## 本章小结

本章我们学习了函数和可调用对象的核心知识：

1. **函数重载与默认参数**：同一个函数名可以对应多个参数列表；默认参数让常用参数可省略，但必须排在参数列表右侧。
2. **`constexpr` 函数**：能在编译期求值，适合常量计算；C++14 后已允许循环和局部变量，C++23 进一步放宽。
3. **Lambda 表达式**：匿名函数语法 `[捕获](参数) -> 返回类型 { 体 }`；支持值/引用捕获、`mutable`、泛型参数、初始化捕获、模板参数（C++20）等。
4. **`std::function` 与 `std::move_only_function`**：前者包装任意可调用对象但要求可复制；后者（C++23）专门支持只能移动的可调用对象。
5. **`std::bind_front/back`**：预先绑定部分参数，产生新的可调用对象。
6. **完美转发与引用折叠**：`T&&` 在模板里是转发引用，能左右通吃；`std::forward` 恢复原值类别；`std::move` 无条件转右值。

下一章我们将进入“类、对象与资源管理”，学习如何自定义类型、管理对象生命周期，以及智能指针和 RAII。

## 练习

1. 写一个重载函数 `print`，分别接受 `int`、`double`、`std::string`，并在 `main` 里分别调用，观察编译器选中哪个版本。
2. 给 `print` 加上默认参数，使得 `print(5)` 和 `print(5, 10)` 都能编译通过。
3. 写一个 Lambda，值捕获一个 `int` 并打印；再写一个引用捕获同名变量的 Lambda，在 Lambda 里修改它，观察外面的变量是否变化。
4. 写一个 `std::function<void(int)>`，分别赋予普通函数、Lambda、仿函数，统一调用。
5. 写一个模板函数 `forwarder`，接受 `T&&` 参数并用 `std::forward` 转发给另一个有左右值重载的函数，分别传入左值和右值，验证转发是否正确。
