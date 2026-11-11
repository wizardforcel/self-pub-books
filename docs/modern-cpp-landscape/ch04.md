# 第 4 章 类、对象与资源管理

如果变量是程序的“名字”，那类就是“图纸”——它描述了一类东西长什么样、能做什么。对象是根据这张图纸制造出来的实体。C++ 是一门面向对象的语言，但它与众不同的地方在于：**C++ 把资源管理（内存、文件句柄、锁、网络连接等）和对象的生命周期绑在了一起**，这被称为 RAII（Resource Acquisition Is Initialization，资源获取即初始化）。

本章我们学习：如何定义类、特殊成员函数（构造、析构、拷贝、移动）、如何用 `= default` / `= delete` 控制生成、委托构造和继承构造、三种智能指针的所有权模型、RAII 在锁和句柄上的扩展，以及 `variant` 和 `any`。

---

## 4.1 特殊成员函数与三五法则的现代简化

### 4.1.1 什么是“特殊成员函数”

C++ 里有六个函数，编译器在特定条件下会自动为类生成，它们叫**特殊成员函数（special member functions）**：

1. 默认构造函数：`Class()`；
2. 析构函数：`~Class()`；
3. 拷贝构造函数：`Class(const Class&)`；
4. 拷贝赋值运算符：`operator=(const Class&)`；
5. 移动构造函数：`Class(Class&&)`；
6. 移动赋值运算符：`operator=(Class&&)`。

其中前四个是“老三四”，C++11 后加入了移动构造和移动赋值，变成“老五六”。你不需要手写它们，编译器会根据你的类里已有的成员自动推断生成。

### 4.1.2 三法则与五法则

在 C++11 之前，有一条著名的“三法则（Rule of Three）”：

> 如果你手动定义了以下三个函数中的任何一个，就应该把另外两个也都定义：
> - 析构函数；
> - 拷贝构造函数；
> - 拷贝赋值运算符。

理由很简单：如果你的类管理某种资源（比如用 `new` 分配了内存），拷贝时就不能简单地“按成员拷贝”，否则两个对象会共享同一块内存，析构时会释放两次，导致崩溃。

C++11 引入移动语义后，这条法则升级成了**五法则（Rule of Five）**：

> 如果你手动定义了析构函数、拷贝构造、拷贝赋值、移动构造、移动赋值中的任何一个，最好把另外四个都考虑清楚。

这五个函数共同决定了对象怎么被创建、拷贝、移动、销毁。

### 4.1.3 `= default` 与 `= delete`

C++11 还引入了两个特殊语法：

- `= default`：告诉编译器“我要默认版本”。用在特殊成员函数上，显式请求编译器生成默认实现。
- `= delete`：告诉编译器“禁用这个函数”。用在特殊成员函数上，可以阻止对象被拷贝、移动，甚至阻止默认构造。

看一个例子：

```cpp
#include <iostream>
#include <string>

class Widget {
public:
    Widget() = default;                    // 显式使用默认构造函数
    Widget(const Widget&) = delete;        // 禁止拷贝构造
    Widget& operator=(const Widget&) = delete;  // 禁止拷贝赋值
    Widget(Widget&&) = default;            // 允许移动构造
    Widget& operator=(Widget&&) = default;       // 允许移动赋值
    ~Widget() = default;                   // 默认析构

    void set_name(std::string n) { name = std::move(n); }
    void print() const { std::cout << name << '\n'; }

private:
    std::string name;
};

int main() {
    Widget w1;
    w1.set_name("原件");
    
    // Widget w2 = w1;      // 编译错误：拷贝被禁止
    Widget w2 = std::move(w1);  // 可以：移动构造
    
    w2.print();
}
```

用 `= delete` 禁用拷贝的常见场景是：类里面包含不应该共享的资源，比如文件句柄、网络连接、`std::unique_ptr` 等。

> **小结**：特殊成员函数决定对象的生命周期管理；三法则/五法则是管理资源的类的金科玉律；`= default` 请求默认实现，`= delete` 显式禁用。

---

## 4.2 构造、委托构造与继承构造

### 4.2.1 构造函数的基本写法

构造函数的名字和类名相同，没有返回类型，用于在对象创建时初始化成员：

```cpp
#include <iostream>
#include <string>

class Person {
public:
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "Person 构造： " << name << '\n';
    }

    void greet() const {
        std::cout << "你好，我是 " << name << "，今年 " << age << " 岁。\n";
    }

private:
    std::string name;
    int age;
};

int main() {
    Person p("张三", 25);
    p.greet();
}
```

这里 `: name(n), age(a)` 是**成员初始化列表**，在构造函数体执行之前就完成了成员的初始化。对于 `std::string` 这种有构造函数的类型，必须用初始化列表，不能在函数体里赋值。

### 4.2.2 委托构造：构造函数调用构造函数

C++11 允许一个构造函数调用同类的另一个构造函数，这叫**委托构造（delegating constructor）**。可以避免重复初始化代码：

```cpp
#include <iostream>
#include <string>

class Rectangle {
public:
    Rectangle(double w, double h) : width(w), height(h) {
        std::cout << "主构造函数\n";
    }

    // 委托构造：正方形，宽高相等
    Rectangle(double side) : Rectangle(side, side) {
        std::cout << "委托构造（正方形）\n";
    }

    // 委托构造：默认 1x1
    Rectangle() : Rectangle(1.0, 1.0) {
        std::cout << "委托构造（默认）\n";
    }

    void print() const {
        std::cout << width << " x " << height << '\n';
    }

private:
    double width;
    double height;
};

int main() {
    Rectangle r1(3, 4);
    r1.print();
    
    Rectangle r2(5);     // 正方形
    r2.print();
    
    Rectangle r3;        // 默认 1x1
    r3.print();
}
```

输出：

```
主构造函数
3 x 4
主构造函数
委托构造（正方形）
5 x 5
主构造函数
委托构造（默认）
1 x 1
```

注意：委托构造的初始化列表里只能写对另一个构造函数的调用，不能同时初始化其他成员。而且要防止**循环委托**（A 委托 B，B 委托 A），否则会无限递归，编译器通常能在编译期或链接期发现。

### 4.2.3 继承构造：让子类复用父类构造函数

如果子类不需要在构造时做额外的初始化，只是想把父类的构造函数“原样继承”，可以用**继承构造（inheriting constructor）**：

```cpp
#include <iostream>
#include <string>

class Animal {
public:
    Animal(const std::string& name) : name(name) {
        std::cout << "Animal 构造：" << name << '\n';
    }

    void speak() const {
        std::cout << name << " 发出声音\n";
    }

protected:
    std::string name;
};

class Dog : public Animal {
public:
    using Animal::Animal;   // 继承父类的构造函数
};

int main() {
    Dog d("旺财");  // 调用继承来的 Animal(const std::string&)
    d.speak();
}
```

`using Animal::Animal;` 这句话把 `Animal` 的所有构造函数带入 `Dog`，`Dog d("旺财")` 会直接调用 `Animal` 的字符串构造函数。子类如果想新增自己的构造函数，也可以和继承构造共存。

> **小结**：构造函数用成员初始化列表初始化成员；委托构造让一个构造函数复用另一个，减少重复代码；继承构造让子类无痛复用父类的构造函数。

---

## 4.3 智能指针：`unique_ptr`、`shared_ptr`、`weak_ptr` 的所有权语义

### 4.3.1 裸指针的问题

C++ 里用 `new` 分配的内存，必须用 `delete` 释放。但如果程序在中间抛出异常、提前返回，或者程序员单纯忘了写 `delete`，就会发生**内存泄漏**。

更糟糕的是，如果两个地方都持有同一个裸指针，都试图 `delete`，就会发生**双重释放**，程序崩溃。

智能指针就是为了解决这些问题而生的：它们把“原始指针”包装成一个对象，在析构时自动释放内存，不用程序员手动 `delete`。

### 4.3.2 `std::unique_ptr`：独占所有权

`std::unique_ptr` 表示**独占所有权**：同一时刻，只有一个 `unique_ptr` 拥有这块内存。它不能复制，只能移动。

```cpp
#include <iostream>
#include <memory>
#include <string>

class Resource {
public:
    Resource(std::string n) : name(std::move(n)) {
        std::cout << name << " 被创建\n";
    }
    ~Resource() {
        std::cout << name << " 被销毁\n";
    }
    void say() const { std::cout << "我是 " << name << '\n'; }
private:
    std::string name;
};

int main() {
    std::unique_ptr<Resource> r1 = std::make_unique<Resource>("独占资源");
    r1->say();

    // std::unique_ptr<Resource> r2 = r1;   // 编译错误：unique_ptr 不能复制
    std::unique_ptr<Resource> r2 = std::move(r1);  // 所有权转移
    
    // r1 已经空了，不能再用
    if (r1) {
        r1->say();
    }
    r2->say();
}  // r2 析构，资源自动释放
```

输出：

```
独占资源 被创建
我是 独占资源
我是 独占资源
独占资源 被销毁
```

关键点：
- `std::make_unique<Resource>(...)` 是创建 `unique_ptr` 的推荐方式（C++14），比直接 `new` 更安全。
- `unique_ptr` 不能复制，只能移动，所有权一经转移，原指针变空。
- `unique_ptr` 析构时自动 `delete` 内存，无需手动释放。

### 4.3.3 `std::shared_ptr`：共享所有权

有时候一块内存需要被多个地方共享，谁最后用完谁释放。`std::shared_ptr` 实现了**共享所有权**：内部维护一个“控制块”，记录有多少个 `shared_ptr` 指向同一块内存，这个计数叫**引用计数**。最后一个 `shared_ptr` 销毁时，内存才会被释放。

```cpp
#include <iostream>
#include <memory>
#include <string>

class Resource {
public:
    Resource(std::string n) : name(std::move(n)) {
        std::cout << name << " 被创建\n";
    }
    ~Resource() {
        std::cout << name << " 被销毁\n";
    }
private:
    std::string name;
};

int main() {
    {
        auto r1 = std::make_shared<Resource>("共享资源");
        std::cout << "引用计数： " << r1.use_count() << '\n';
        
        {
            auto r2 = r1;  // 拷贝，引用计数加 1
            std::cout << "引用计数： " << r1.use_count() << '\n';
        }  // r2 销毁，引用计数减 1
        
        std::cout << "引用计数： " << r1.use_count() << '\n';
    }  // r1 销毁，引用计数归零，资源释放
}
```

输出：

```
共享资源 被创建
引用计数： 1
引用计数： 2
引用计数： 1
共享资源 被销毁
```

`shared_ptr` 可以复制，每复制一次引用计数加 1，每销毁一次减 1。当引用计数归零时，自动释放内存。

`shared_ptr` 的代价是：控制块需要额外的内存，引用计数的增减是原子操作（线程安全但有开销）。如果不需要共享，优先用 `unique_ptr`。

### 4.3.4 `std::weak_ptr`：打破循环引用的观察者

`shared_ptr` 有一个经典问题：**循环引用**。如果对象 A 持有 B 的 `shared_ptr`，B 又持有 A 的 `shared_ptr`，它们的引用计数永远不会归零，内存就泄漏了。

`std::weak_ptr` 是为了解决这个问题而设计的。它是 `shared_ptr` 的“观察者版”：它可以观察一块由 `shared_ptr` 管理的内存，但不增加引用计数。要访问内存，需要先把 `weak_ptr` 锁成 `shared_ptr`：

```cpp
#include <iostream>
#include <memory>

class Node {
public:
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;   // 用 weak_ptr 打破循环
    
    ~Node() {
        std::cout << "Node 销毁\n";
    }
};

int main() {
    auto a = std::make_shared<Node>();
    auto b = std::make_shared<Node>();
    
    a->next = b;
    b->prev = a;   // weak_ptr，不增加 a 的引用计数
    
    std::cout << "a 的引用计数： " << a.use_count() << '\n';  // 1，不是 2
    
    // 通过 weak_ptr 访问
    if (auto p = b.prev.lock()) {
        std::cout << "能通过 prev 访问前驱节点\n";
    }
}
```

`weak_ptr` 的常用方法：
- `lock()`：如果原内存还活着，返回一个 `shared_ptr`；否则返回空的 `shared_ptr`。
- `expired()`：判断原内存是否已被释放。

### 4.3.5 三种智能指针怎么选

| 智能指针 | 所有权模型 | 能否复制 | 典型场景 |
|---|---|---|---|
| `std::unique_ptr` | 独占 | 不能复制，只能移动 | 独占资源、工厂函数返回、pimpl 惯用法 |
| `std::shared_ptr` | 共享 | 可以复制，引用计数 | 多个对象共享同一资源、图结构中的节点 |
| `std::weak_ptr` | 观察（不拥有） | 可以复制 | 配合 `shared_ptr` 打破循环引用、缓存 |

选型口诀：**默认用 `unique_ptr`；需要共享才用 `shared_ptr`；担心循环引用就配 `weak_ptr`；尽量避免裸 `new` / `delete`。**

> **小结**：`unique_ptr` 独占所有权、不可复制；`shared_ptr` 共享所有权、引用计数；`weak_ptr` 观察但不拥有，配合 `shared_ptr` 打破循环引用。三者构成现代 C++ 内存管理的基础。

---

## 4.4 RAII 的现代扩展：锁守卫与资源句柄

### 4.4.1 什么是 RAII

**RAII（Resource Acquisition Is Initialization，资源获取即初始化）** 是 C++ 资源管理的核心思想：

- 把资源的获取绑定到对象的初始化（构造函数）；
- 把资源的释放绑定到对象的销毁（析构函数）。

只要对象在栈上，离开作用域时析构函数一定会被调用，资源就一定会被释放。即使函数中间抛出异常、提前返回，栈上的对象也会被正确销毁，资源不会泄漏。

智能指针就是 RAII 的典型应用：构造时 `new`，析构时 `delete`。

### 4.4.2 锁守卫：`std::lock_guard` 与 `std::unique_lock`

互斥量（mutex）用于保护共享资源，防止多个线程同时修改。手动 lock/unlock 很危险：如果中间抛出异常，unlock 可能执行不到，导致死锁。

`std::lock_guard` 是 RAII 风格的锁守卫：构造时加锁，析构时解锁：

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int counter = 0;
std::mutex mtx;

void increment(int id) {
    std::lock_guard<std::mutex> lock(mtx);  // 构造时加锁
    ++counter;
    std::cout << "线程 " << id << " 修改 counter，当前值： " << counter << '\n';
    // 析构时自动解锁，即使这里抛出异常也安全
}

int main() {
    std::thread t1(increment, 1);
    std::thread t2(increment, 2);
    t1.join();
    t2.join();
}
```

`std::unique_lock` 功能更丰富：可以延迟加锁、手动 unlock、支持条件变量等。`std::scoped_lock`（C++17）可以一次性锁住多个互斥量，避免死锁。

### 4.4.3 资源句柄：文件、句柄的 RAII 包装

除了内存和锁，文件句柄、网络套接字、图形句柄等都可以用 RAII 包装。例如 `std::ofstream` 在析构时会自动关闭文件：

```cpp
#include <fstream>
#include <string>

void write_file(const std::string& path) {
    std::ofstream out(path);
    out << "Hello, RAII\n";
    // out 析构时自动关闭文件，不用手动调用 out.close()
}
```

如果你自己管理操作系统资源（比如 Windows 的 `HANDLE`、POSIX 的文件描述符），也可以写一个小类，在构造时打开资源，析构时关闭资源，这就是“资源句柄”的 RAII 模式。

> **小结**：RAII 的核心是“构造获取资源，析构释放资源”；`lock_guard` / `unique_lock` 把互斥锁管理自动化；`ofstream` 等标准类也遵循 RAII；自定义资源句柄也应遵循这一原则。

---

## 4.5 联合体、`std::variant` 与 `std::any`

### 4.5.1 联合体 `union`：同一块内存的不同解释

**联合体（union）** 是一种特殊的类类型：它的所有成员共享同一块内存，任意时刻只能有一个成员有效。联合体的大小等于其最大成员的大小。

```cpp
#include <iostream>

union Data {
    int i;
    double d;
    char c;
};

int main() {
    Data data;
    data.i = 42;
    std::cout << data.i << '\n';   // 42
    
    data.d = 3.14;
    std::cout << data.d << '\n';   // 3.14
    // std::cout << data.i << '\n';  // 危险：读取上一个成员，未定义行为
}
```

联合体节省内存，但不安全：程序员必须自己记得当前存的是哪种类型，读错就会未定义行为。现代 C++ 中，`std::variant` 是更安全的替代品。

### 4.5.2 `std::variant`：类型安全的联合体（C++17）

`std::variant` 表示“一个值，可能是多种类型中的一种”，并且知道当前到底是哪一种。它是**封闭集合**：你必须在创建时指定所有可能的类型。

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, double, std::string> v;
    
    v = 42;                // 现在 v 存的是 int
    std::cout << std::get<int>(v) << '\n';
    
    v = 3.14;              // 现在 v 存的是 double
    std::cout << std::get<double>(v) << '\n';
    
    v = std::string("hello");  // 现在 v 存的是 string
    std::cout << std::get<std::string>(v) << '\n';
    
    // 访问错误的类型会抛出 std::bad_variant_access
    // std::cout << std::get<int>(v) << '\n';  // 运行期异常
}
```

`std::variant` 的常用工具：
- `std::get<T>(v)`：取出某个类型的值，类型不对抛异常；
- `std::get_if<T>(&v)`：安全地尝试取，不对返回 `nullptr`；
- `std::holds_alternative<T>(v)`：判断当前是不是类型 `T`；
- `std::visit`：根据当前类型调用不同的处理逻辑（像模式匹配）。

`std::visit` 示例：

```cpp
std::variant<int, double, std::string> v = 42;

std::visit([](auto&& arg) {
    std::cout << "类型是 " << typeid(arg).name() << ", 值是 " << arg << '\n';
}, v);
```

### 4.5.3 `std::any`：运行期任意类型（C++17）

如果 `std::variant` 是“封闭集合”（类型在编译期就限定死了），那 **`std::any`** 就是“开放集合”——它可以存放任何类型的值，类型信息在运行期才知道。

```cpp
#include <iostream>
#include <any>
#include <string>

int main() {
    std::any a = 42;
    std::cout << std::any_cast<int>(a) << '\n';
    
    a = std::string("hello");
    std::cout << std::any_cast<std::string>(a) << '\n';
    
    a = 3.14;
    if (a.type() == typeid(double)) {
        std::cout << std::any_cast<double>(a) << '\n';
    }
}
```

`std::any_cast` 取值，类型不对抛 `std::bad_any_cast`。`any` 比 `variant` 更灵活，但也更重：需要类型擦除、运行期 RTTI 支持，性能开销更大。

### 4.5.4 三者选型

| 设施 | 类型集合 | 安全性 | 性能 | 适用场景 |
|---|---|---|---|---|
| `union` | 程序员自定 | 低（需手动跟踪类型） | 极高 | 极端性能敏感、C 兼容 |
| `std::variant` | 编译期封闭 | 高（类型错误抛异常） | 高 | 有限且已知的多种类型 |
| `std::any` | 运行期任意 | 中（`any_cast` 可能抛异常） | 较低 | 类型完全未知、动态场景 |

> **小结**：`union` 是最底层的内存共享，不安全；`std::variant` 是类型安全的封闭联合体，适合已知类型集合；`std::any` 是运行期任意类型容器，灵活但开销大。优先 `variant`，只有类型完全不可知时才用 `any`。

---

## 本章小结

本章我们学习了类与资源管理的核心知识：

1. **特殊成员函数**：默认构造、析构、拷贝/移动构造与赋值，编译器可自动生成；三法则/五法则指导资源类的设计。
2. **`= default` / `= delete`**：显式请求默认实现或禁用某个函数，常用于禁止拷贝。
3. **构造函数**：成员初始化列表、委托构造（减少重复）、继承构造（子类复用父类构造函数）。
4. **智能指针**：`unique_ptr` 独占所有权、`shared_ptr` 共享所有权（引用计数）、`weak_ptr` 观察不拥有（打破循环引用）。
5. **RAII**：构造获取资源、析构释放资源，适用于内存、锁（`lock_guard`）、文件句柄等。
6. **`variant` / `any` / `union`**：从底层内存共享到类型安全的封闭联合体，再到运行期任意类型，按需选用。

下一章我们将进入“模板、泛型与编译期抽象”，学习函数模板、类模板、变量模板，以及 C++20 的概念与约束。

## 练习

1. 定义一个类 `FileHandle`，在构造时打开一个文件（`std::ofstream`），析构时自动关闭，禁止拷贝但允许移动。
2. 用 `std::unique_ptr` 管理一个动态分配的 `int` 数组，并用 `std::move` 转移所有权。
3. 写一个结构体，里面同时包含 `std::shared_ptr` 和 `std::weak_ptr` 指向同类型的另一个对象，验证不会发生循环引用泄漏。
4. 用 `std::variant<int, std::string>` 存储不同类型的值，用 `std::visit` 打印它们，再故意用 `std::get` 访问错误类型，观察异常。
5. 用 `std::lock_guard` 保护一个全局计数器，启动两个线程同时累加，验证结果正确。
