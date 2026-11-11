# 《现代 C++ 全景：语言特性、标准库与工程化实践》细纲（终版）

## 全书结构概览

| 篇 | 主题 | 章范围 |
|---|---|---|
| 第一篇 | 现代 C++ 基础与语言核心演进（含 c99sh 脚本化开发与 CMake/vcpkg 工程化基础） | 第 1–7 章 |
| 第二篇 | 标准库常用基础设施（含完整并发支持库） | 第 8–12 章 |
| 第三篇 | 标准库底层支撑与计算专项 | 第 13–21 章 |
| 第四篇 | Boost 准标准库精要 | 第 22–28 章 |
| 第五篇 | 工程实践、跨版本演进与附录 | 第 29–31 章 + 附录 |

> 全书贯穿两条工具主线：**第一篇至第 7 章之前**，所有单文件示例均通过 c99sh（以 `cxxsh` 调用）像脚本一样执行，读者无需接触编译器命令行；**第 7 章之后**，示例统一切换到 CMake + vcpkg 的工程化构建模板。


# 第一篇：现代 C++ 基础与语言核心演进

本篇建立现代 C++ 的统一语言观。C++11 之后，值类别、类型推导、资源生命周期和编译期计算被重新设计，后续标准都在这一新地基上迭代。本篇按“基础机制 → 声明与初始化 → 函数与可调用对象 → 类与资源管理 → 模板与编译期抽象 → 模块与错误处理 → 工程化构建”的顺序组织，并在第一章末尾用 c99sh 带领读者完成第一个可运行项目。

## 第 1 章 现代 C++ 的定位与工具链

### 1.1 从 C++11 到 C++26：标准演进脉络与兼容性策略
介绍各标准版本的定位差异、特性测试宏和编译器支持矩阵的读法，帮助读者建立跨版本写作与选型意识。**对应**：`__cplusplus`、`__cpp_*` 特性测试宏（C++20）、编译器支持表。

### 1.2 编译、链接与翻译阶段：现代构建的最小知识
介绍预处理、翻译单元、链接模型，以及头文件与模块在物理组织上的根本差异；本节只建立概念模型，不要求读者手动执行编译链接命令。**对应**：翻译阶段、`#include` 模型、模块接口单元（C++20）。

### 1.3 类型系统鸟瞰：基本类型、复合类型与类型属性
梳理算术类型、指针、引用、数组、函数类型和 `decltype` 推导规则，建立类型分类的全局地图。**对应**：基本类型、`decltype`（C++11）、`std::is_*` 类型特征（C++11）。

### 1.4 值类别与对象生命周期：理解移动、转发和所有权的钥匙
以 lvalue/prvalue/xvalue 为主线，串联临时对象实质化、复制消除、引用折叠和 `std::move` 的准确语义。**对应**：值类别、复制消除（C++17）、右值引用（C++11）、`std::move` / `std::forward`。

### 1.5 第一个项目：用 c99sh 像脚本一样运行 Hello World
本节用 c99sh 把“写源码 → 运行”压缩成一步，让读者在几秒钟内看到第一个 C++ 程序的输出，而不被编译、链接的命令行细节拖慢节奏。c99sh 的设计初衷是让单文件 C/C++ 程序像脚本一样被执行：它内部完成编译和链接，对使用者透明；C++ 支持通过以 `cxxsh` 为名调用 c99sh 来启用。第一篇所有单文件示例都建立在这个工具上，直到第 7 章讲完 CMake 后再切换。**对应**：c99sh / cxxsh（单文件 C++ 脚本执行）、shebang（`#!/usr/bin/env cxxsh`）、rcfile（`~/.cxxshrc` / 项目 `cxxshrc`，配置 `-std=`、警告选项与常用 `#include`）、`std::string_view`（C++17）、`constexpr`（C++11）、`std::format`（C++20）/ `std::println`（C++23）、`main` 返回值。

## 第 2 章 声明、初始化与作用域

### 2.1 `auto`、`decltype` 与返回类型推导
说明类型推导在变量、返回类型和泛型代码中的不同规则，重点区分 `auto` 与 `decltype(auto)`。**对应**：`auto`（C++11）、`decltype`（C++11）、`decltype(auto)`（C++14）、返回类型推导。

### 2.2 统一初始化与列表初始化
辨析 `{}` 初始化在直接初始化、复制初始化、窄化检查和 `initializer_list` 构造函数优先级中的行为。**对应**：列表初始化（C++11）、`std::initializer_list`（C++11）、聚合初始化增强。

### 2.3 结构化绑定与聚合体
介绍如何用结构化绑定解构 pair、tuple、数组和聚合体，以及聚合体定义的现代放宽。**对应**：结构化绑定（C++17）、聚合体初始化。

### 2.4 命名空间、内联命名空间与嵌套命名空间简化
说明命名空间在库版本控制和 API 演进中的工程用法。**对应**：嵌套命名空间定义（C++17）、内联命名空间。

### 2.5 `const`、`constexpr`、`consteval` 与 `constinit`
按“运行期只读 → 编译期可求值 → 编译期必须求值 → 编译期初始化”厘清四个限定符的边界。**对应**：`constexpr`（C++11/14 放宽）、`consteval`（C++20）、`constinit`（C++20）。

## 第 3 章 函数、Lambda 与可调用对象

### 3.1 函数重载、默认参数与 `constexpr` 函数
梳理重载决议基本规则，以及 `constexpr` 函数从严格限制到逐步放宽的演进。**对应**：函数重载、默认参数、放宽的 `constexpr`（C++14）、`constexpr` 函数中的 `goto` 与标签（C++23）。

### 3.2 Lambda 表达式：从 C++11 到 C++23 的完整形态
覆盖捕获列表、`mutable`、初始化捕获、泛型 lambda、模板 lambda、`*this` 捕获和省略 `()`。**对应**：Lambda（C++11）、初始化捕获（C++14）、泛型 lambda（C++14）、模板参数列表 lambda（C++20）、省略 `()`（C++23）。

### 3.3 `std::function`、`std::move_only_function` 与 `std::bind_front/back`
比较类型擦除可调用包装器的适用边界，以及仅移动可调用对象在异步和资源管理中的必要性。**对应**：`std::function`（C++11）、`std::move_only_function`（C++23）、`std::bind_front`（C++20）、`std::bind_back`（C++23）。

### 3.4 完美转发与引用折叠
以转发引用和引用折叠规则解释 `std::forward` 的推导机制及其在泛型工厂、`emplace` 和包装器中的应用。**对应**：转发引用、引用折叠、`std::forward`。

## 第 4 章 类、对象与资源管理

### 4.1 特殊成员函数与三五法则的现代简化
说明移动构造/移动赋值如何改变 Rule of Five，以及 `= default` / `= delete` 的控制作用。**对应**：移动语义（C++11）、特殊成员函数、`= default` / `= delete`。

### 4.2 构造、委托构造与继承构造
介绍委托构造和继承构造如何减少样板代码，以及初始化和异常安全注意事项。**对应**：委托构造函数（C++11）、继承构造函数（C++11）。

### 4.3 智能指针：`unique_ptr`、`shared_ptr`、`weak_ptr` 的所有权语义
以所有权模型讲解三种指针，重点说明控制块、循环引用和观察者角色。**对应**：`std::unique_ptr` / `shared_ptr` / `weak_ptr`（C++11）、`std::make_unique`（C++14）。

### 4.4 RAII 的现代扩展：锁守卫与资源句柄
把 RAII 从内存管理推广到锁管理和文件句柄，说明“析构即释放”为何是现代 C++ 资源安全的统一策略；线程生命周期相关的 RAII 设施在第二篇并发章展开。**对应**：`std::lock_guard` / `unique_lock`（C++11）、RAII。

### 4.5 联合体、`std::variant` 与 `std::any`
讨论类型安全联合体替代方案，比较 `variant` 的封闭集合与 `any` 的运行期任意类型。**对应**：`union`、`std::variant`（C++17）、`std::any`（C++17）。

## 第 5 章 模板、泛型与编译期抽象

### 5.1 函数模板、类模板与变量模板
从基础模板形式出发，强调变量模板在编译期常量和类型特征萃取中的简洁表达。**对应**：函数模板、类模板、变量模板（C++14）。

### 5.2 模板参数推导与 CTAD
说明类模板参数推导如何让 pair、tuple、容器适配器不再需要显式模板实参。**对应**：CTAD（C++17）、推导指引。

### 5.3 可变参数模板与折叠表达式
以 `sizeof...`、包展开和折叠表达式为主线，展示参数包的线性处理方式。**对应**：可变参数模板（C++11）、折叠表达式（C++17）。

### 5.4 `if constexpr` 与编译期分支
解释 `if constexpr` 如何替代标签分发和 SFINAE 实现编译期分支。**对应**：`if constexpr`（C++17）。

### 5.5 概念与约束：从 SFINAE 到 `concepts`
覆盖 `concept` 定义、`requires` 子句、简写模板、约束偏序与合取/析取，并给出迁移对照。**对应**：概念与约束（C++20）、`std::concepts` 库。

### 5.6 模板元编程的现代替代：`type_traits`、`integer_sequence` 与 `ratio`
说明大多数传统 TMP 需求已被标准类型特征和编译期序列/有理数设施覆盖。**对应**：`<type_traits>`（C++11）、`std::integer_sequence`（C++14）、`std::ratio`（C++11）。

## 第 6 章 模块、契约与错误处理模型

### 6.1 模块：接口单元、实现单元与 `import`
说明模块如何替代头文件展开、控制宏泄漏、缩短编译时间。**对应**：模块（C++20）、`import` / `export`。

### 6.2 异常、`std::error_code` 与 `std::expected`
按“异常 / 系统错误码 / 类型安全返回值或错误”的层次给出错误处理选择框架。**对应**：异常（C++11 增强）、`std::error_code`（C++11）、`std::expected`（C++23）。

### 6.3 契约（C++26 前瞻）：`pre`、`post`、`contract_assert`
介绍契约断言如何把前置条件、后置条件和断言内建于声明。**对应**：契约（C++26）、`contract_assert`。

### 6.4 `std::source_location` 与诊断信息
说明如何在不使用宏的情况下获取文件名、行号和函数名。**对应**：`std::source_location`（C++20）。

## 第 7 章 CMake 与 vcpkg：现代 C++ 工程化基础

本章是全书工程工具的分水岭：从本章起，示例项目告别 cxxsh 的单文件脚本模式，进入真实项目的工程化构建。本章目标不是培养 CMake 专家，而是让读者写出目标化、可复现、可移植的构建脚本，并用 vcpkg 以声明式方式管理第三方依赖。后续所有章节的示例均默认使用本章建立的工程模板。

### 7.1 最小现代 CMake 工程：从 `cmake_minimum_required` 到可执行文件
以一个最小 `CMakeLists.txt` 为起点，说明现代 CMake 的基本结构：版本要求、`project()`、C++ 标准设置、目标创建与构建目录约定，并完成从 cxxsh 脚本到 CMake 项目的迁移。**对应**：`cmake_minimum_required`、`project`、`add_executable` / `add_library`、`CMAKE_CXX_STANDARD` / `CMAKE_CXX_STANDARD_REQUIRED` / `CMAKE_CXX_EXTENSIONS`。

### 7.2 目标化构建：`target_*` 命令与 PUBLIC / PRIVATE / INTERFACE
以“属性附着于目标而非目录”为主线，讲解目标级命令并系统说明三种传播范围的语义差异，强调避免目录级命令。**对应**：`target_include_directories`、`target_compile_definitions`、`target_compile_options`、`target_link_libraries`、`target_compile_features`、`target_sources`、`PUBLIC` / `PRIVATE` / `INTERFACE`。

### 7.3 生成器表达式：编译期条件化的目标属性
介绍 `$<...>` 生成器表达式在构建系统生成期求值的机制，以及按编译器 ID 区分警告选项、区分构建树与安装树头文件路径等常用场景。**对应**：生成器表达式（`$<CXX_COMPILER_ID>`、`$<BUILD_INTERFACE>`、`$<INSTALL_INTERFACE>`、`$<CONFIG>`）。

### 7.4 第三方依赖的三种获取方式：`find_package`、`FetchContent` 与 `add_subdirectory`
比较三种依赖获取机制的适用边界：系统或包管理器已安装的库、一键构建场景、vendored 依赖。**对应**：`find_package`、`FetchContent_MakeAvailable`、`add_subdirectory`。

### 7.5 vcpkg 集成：工具链文件、清单模式与 `vcpkg.json`
说明 vcpkg 通过 CMake 工具链文件透明注入包搜索路径，以及清单模式下 `vcpkg.json` 如何声明直接依赖并在配置阶段自动安装。**对应**：vcpkg 工具链文件、清单模式、`vcpkg.json`（`dependencies`、`features`、`default-features`、`platform`）。

### 7.6 `CMakePresets.json`：可复现的配置、构建与测试预设
介绍用 `CMakePresets.json` 固化生成器、构建类型、缓存变量和依赖路径，并用 `CMakeUserPresets.json` 允许开发者本地覆盖。**对应**：`CMakePresets.json`（`configurePresets` / `buildPresets` / `testPresets`）、`cmake --preset` / `cmake --build --preset` / `ctest --preset`。

### 7.7 完整工程模板：从零到可构建、可测试、可安装
综合本章所有设施，给出包含库目标、可执行目标、vcpkg 依赖、预设配置和基础安装规则的完整项目骨架，作为后续各章示例的工程基线。**对应**：综合运用 `target_*` 命令、生成器表达式、`find_package` / vcpkg 清单、`CMakePresets.json`、`install()` / `export()`。


# 第二篇：标准库常用基础设施

本篇面向应用开发最常用的标准库设施，按“通用工具 → 数据组织 → 文本与正则 → 系统 I/O → 并发执行”的顺序展开。并发章完整覆盖从线程生命周期、锁与条件变量、同步原语、原子与内存序，到原子智能指针、RCU 和协程/异步执行的设施。

## 第 8 章 通用工具库

### 8.1 `pair`、`tuple` 与 `apply` / `make_from_tuple`
覆盖 tuple 的构造、访问、拼接和与 `apply` 的配合，以及结构化绑定的自然使用。**对应**：`std::pair`、`std::tuple`（C++11）、`std::apply`（C++17）、`std::make_from_tuple`（C++17）。

### 8.2 `optional`、`variant`、`expected` 的词汇类型设计
比较三种“可能缺失 / 多态 / 错误”的词汇类型在接口表达力、存储开销和单子操作上的差异。**对应**：`std::optional`（C++17）、`std::variant`（C++17）、`std::expected`（C++23）、单子操作（C++23）。

### 8.3 `bitset`、位操作与 `std::byteswap`
介绍固定大小位序列、C++20 位操作函数和 C++23 字节交换。**对应**：`std::bitset`、位操作（C++20）、`std::byteswap`（C++23）。

### 8.4 `std::hash`、函数对象与引用包装器
说明标准哈希设施的使用与自定义方式，以及 `std::ref` / `std::cref` 的引用语义保持。**对应**：`std::hash`（C++11）、`std::reference_wrapper`（C++11）。

## 第 9 章 容器库

### 9.1 序列容器：`vector`、`array`、`deque`、`list`、`forward_list`
以 `vector` 为默认序列容器展开，说明 `array` 的零开销聚合语义、`deque` 的双端增长特性和链表容器收窄的适用场景。**对应**：`std::vector`、`std::array`（C++11）、`std::deque`、`std::list`、`std::forward_list`（C++11）。

### 9.2 关联容器：有序 `map`/`set` 与无序 `unordered_*`
比较红黑树与哈希表在复杂度、迭代器稳定性、内存局部性和有序性上的权衡，并介绍节点句柄。**对应**：`std::map` / `set` / `multimap` / `multiset`、`std::unordered_*`（C++11）、`extract` / `merge`（C++17）。

### 9.3 容器适配器：`stack`、`queue`、`priority_queue` 与 `flat_map`/`flat_set`
说明适配器的底层容器选择，以及 C++23 扁平容器如何以连续存储换取缓存友好性。**对应**：`stack` / `queue` / `priority_queue`、`flat_map` / `flat_set`（C++23）。

### 9.4 非拥有视图型容器：`span` 与 `mdspan`
介绍非拥有连续视图和多维数组视图的轻量语义，重点说明 `mdspan` 的布局与索引策略。**对应**：`std::span`（C++20）、`std::mdspan`（C++23）。

## 第 10 章 字符串、文本与正则库

### 10.1 `std::string`、`std::string_view` 与字符串操作
比较拥有字符串与视图字符串的所有权、生命周期和性能特征，覆盖现代实用成员。**对应**：`std::basic_string`、`std::basic_string_view`（C++17）、`starts_with` / `ends_with`（C++20）、`resize_and_overwrite`（C++23）。

### 10.2 格式化库：`std::format` 与 `std::print`
系统讲解格式说明符、`std::formatter` 自定义扩展、编译期格式字符串检查，以及 `std::print` 的替代定位。**对应**：格式化库（C++20）、`std::format`、`std::print` / `println`（C++23）。

### 10.3 字符分类、本地化与 Unicode 基础
介绍 `<cctype>`、`<locale>`、宽字符转换设施的现状与弃用情况，以及文本编码识别支持。**对应**：窄/宽字符分类、`<locale>`、文本编码识别（C++23）。

### 10.4 正则表达式库
覆盖 `basic_regex`、匹配/搜索/替换算法、匹配结果迭代器和 ECMAScript 语法，强调编译正则的性能成本。**对应**：`<regex>`（C++11）、`std::basic_regex`、`regex_match` / `regex_search` / `regex_replace`。

## 第 11 章 输入/输出与文件系统库

### 11.1 流式 I/O：`iostream`、格式化与操纵符
覆盖标准流对象、流状态、格式化操纵符，以及 `syncstream` 在并发输出中的同步语义。**对应**：`<iostream>`、`<iomanip>`、`std::basic_osyncstream`（C++20）。

### 11.2 字符串流与 `spanstream`
说明 `stringstream` 的传统用法，以及 `spanstream` 如何基于固定缓冲区提供无分配的流式格式化/解析。**对应**：`<sstream>`、`<spanstream>`（C++23）。

### 11.3 文件系统库
覆盖 `path`、目录遍历、文件状态查询、复制/删除/重命名等操作，以及错误处理重载的选择。**对应**：`<filesystem>`（C++17）、`std::filesystem::path`、目录迭代器。

## 第 12 章 并发支持库

本章完整覆盖并发编程：从线程生命周期与协作取消，到锁、条件变量和同步原语，再到原子操作与内存序；最后深入原子智能指针、危险指针/RCU 以及协程与异步执行框架。它是全书唯一的并发专章，第 4 章的 RAII 和第 7 章的模块化构建为此章提供语言与工程基础。

### 12.1 线程生命周期与协作取消：`thread`、`jthread` 与停止令牌
从手动 `join`/`detach` 的常见缺陷出发，引入 `jthread` 的自动汇合和内置停止状态模型，并深入 `stop_source` / `stop_token` / `stop_callback` 的协作取消机制。**对应**：`std::thread`（C++11）、`std::jthread`（C++20）、`stop_token` 系列（C++20）。

### 12.2 互斥量、条件变量与锁管理
覆盖 `mutex`、`recursive_mutex`、`condition_variable` / `condition_variable_any`，以及 `lock_guard` / `unique_lock` / `scoped_lock` 的锁管理策略和避免死锁的实践。**对应**：互斥量与条件变量（C++11）、`std::scoped_lock`（C++17）、锁守卫（C++11）。

### 12.3 闩锁、屏障与信号量
比较三种 C++20 同步原语：`latch` 一次性计数、`barrier` 可重用阶段性同步、`counting_semaphore` 资源计数，给出选型依据。**对应**：`std::latch` / `barrier` / `counting_semaphore`（C++20）。

### 12.4 原子操作与内存序
介绍 `std::atomic`、`atomic_flag` 和六种内存序的直观含义，说明默认 `seq_cst` 何时足够、何时需要放宽，以及 `atomic::wait/notify` 的等待/通知机制。**对应**：`std::atomic`（C++11）、内存序、`atomic::wait` / `notify_one` / `notify_all`（C++20）。

### 12.5 原子智能指针、危险指针与 RCU（C++26 前瞻）
介绍 `atomic<shared_ptr<T>>` 的无锁安全共享，以及 C++26 危险指针和读-复制-更新机制的方向与适用场景。**对应**：`std::atomic<std::shared_ptr<T>>`（C++20）、危险指针（C++26）、RCU（C++26）。

### 12.6 协程与异步执行：`generator` 与 `std::execution`
以惰性序列生成为首个场景，说明 `co_await` / `co_yield` / `co_return`、协程帧、承诺类型和 awaiter 的协作方式；再介绍 C++23 `std::generator` 和 C++26 `std::execution` 的 sender/receiver 异步执行模型。**对应**：协程（C++20）、`std::coroutine` 支持库、`std::generator`（C++23）、`std::execution`（C++26 前瞻）。


# 第三篇：标准库底层支撑与计算专项

本篇覆盖标准库的底层支撑设施和计算抽象。语言支持、诊断、内存管理、概念与元编程为所有标准库组件提供地基；迭代器、范围、算法、数值和时间则完成“如何遍历、组合、计算和表达”的最后一层。

## 第 13 章 语言支持库

### 13.1 类型属性、固定宽度整数与浮点限制
介绍 `numeric_limits`、`<cstdint>`、固定宽度浮点类型和 C 数值限制头文件的现代用法。**对应**：`std::numeric_limits`、`<cstdint>`、固定宽度浮点类型（C++23）、`<climits>` / `<cfloat>`。

### 13.2 `std::initializer_list`、`std::type_info` 与 `std::type_index`
说明初始化列表在容器和算法中的底层角色，以及 RTTI 在类型擦除和容器键中的受控使用。**对应**：`std::initializer_list`（C++11）、`std::type_info`、`std::type_index`。

### 13.3 实现属性查询与特性测试宏
介绍 `__has_include`、`__cpp_lib_*` 和 `__cpp_*` 如何在跨编译器代码中做条件编译。**对应**：特性测试宏（C++20）、`__has_include`。

## 第 14 章 诊断库

### 14.1 断言、`errno` 与系统错误
覆盖 `assert`、`errno` 的适用边界，以及 `error_code` / `error_condition` 的类型安全错误表达。**对应**：`<cassert>`、`errno`、`std::error_code`（C++11）。

### 14.2 标准异常层次与异常安全
梳理 `std::exception` 派生体系、`what()` 契约，以及 `noexcept` 对移动和容器操作的影响。**对应**：标准异常类型、`noexcept`（C++11）。

### 14.3 `std::stacktrace`：程序化调用栈捕获
说明如何在运行期捕获和遍历调用栈条目，用于日志、崩溃报告和调试工具。**对应**：`std::stacktrace`（C++23）。

## 第 15 章 内存管理库

### 15.1 低级内存管理：`new` / `delete` 与 `std::allocator`
说明分配/释放的基本契约，以及现代代码为何应优先使用智能指针和容器。**对应**：`operator new` / `delete`、`std::allocator`。

### 15.2 多态分配器与 `std::pmr`
介绍 `memory_resource`、`polymorphic_allocator` 和 `monotonic_buffer_resource` 在减少分配开销中的作用。**对应**：多态分配器与内存资源（C++17）、`std::pmr::*`。

### 15.3 未初始化内存算法
覆盖 `uninitialized_copy`、`uninitialized_default_construct` 等在容器和自定义数据结构中的底层用途。**对应**：`<memory>` 未初始化内存算法。

## 第 16 章 概念库与元编程库

### 16.1 基本库概念：`same_as`、`convertible_to`、`integral`、`invocable` 等
逐一介绍标准概念库核心概念，并说明如何用它们替代手写 `enable_if` 条件。**对应**：概念库（C++20）、`<concepts>`。

### 16.2 编译期类型信息与 `std::integral_constant`
介绍 `integral_constant`、`bool_constant`、`true_type` / `false_type` 作为编译期值载体的作用。**对应**：`<type_traits>` 编译期类型信息工具。

### 16.3 编译时有理数运算与 `std::ratio`
说明 `ratio` 在时间间隔和物理量编译期表示中的用法。**对应**：`std::ratio`（C++11）、`ratio_add` / `ratio_less` 等元函数。

## 第 17 章 迭代器库

### 17.1 迭代器类别与 `iterator_traits`
梳理输入、前向、双向、随机访问、连续迭代器的能力阶梯，以及现代代码应优先使用范围算法的原因。**对应**：迭代器类别、`std::iterator_traits`。

### 17.2 迭代器适配器：`reverse_iterator`、`move_iterator`、插入迭代器
介绍常用迭代器适配器如何改变遍历方向或插入语义。**对应**：`reverse_iterator`、`move_iterator`（C++11）、`back_insert_iterator` 等。

### 17.3 范围迭代器与哨兵
说明 C++20 起迭代器与哨兵类型可以不同，从而支持空终止序列和惰性范围。**对应**：范围迭代器（C++20）、哨兵、`common_range`。

## 第 18 章 范围库

### 18.1 范围概念与 `std::ranges` 算法
说明 `range`、`viewable_range`、`borrowed_range` 等概念如何约束算法接口。**对应**：范围库（C++20）、约束算法。

### 18.2 视图与惰性求值：`filter`、`transform`、`take`、`drop`、`join`
以管道 `|` 组合展示视图的惰性、零分配和可组合性，并说明生命周期陷阱。**对应**：范围适配器（C++20）、`views::filter` / `transform` / `take` / `drop` / `join`。

### 18.3 `std::generator` 与范围工厂
介绍 `std::generator` 作为协程驱动的惰性序列，以及 `views::iota`、`views::repeat` 等工厂的配合。**对应**：`std::generator`（C++23）、范围工厂。

### 18.4 `ranges::to` 与范围物化
说明如何将惰性视图转换为实际容器。**对应**：`ranges::to`（C++23）。

## 第 19 章 算法库

### 19.1 非修改序列算法与修改序列算法
覆盖查找、计数、比较、复制、变换、替换、删除等经典算法，强调算法优先于手写循环。**对应**：`<algorithm>` 非修改/修改序列算法。

### 19.2 排序、二分查找与堆算法
介绍 `sort` / `stable_sort` / `partial_sort`、二分查找和堆算法的适用前提与复杂度。**对应**：排序算法、二分查找、堆算法。

### 19.3 数值算法与并行执行策略
覆盖 `accumulate`、`reduce`、`transform_reduce` 等数值算法，以及 C++17 执行策略的使用条件和限制。**对应**：`<numeric>` 数值算法、执行策略（C++17）。

### 19.4 受约束算法（C++20）
说明 `std::ranges` 命名空间下的算法如何消除迭代器对不匹配并提供更清晰编译错误。**对应**：受约束算法（C++20）。

## 第 20 章 数值库

### 20.1 常用数学函数、特殊数学函数与数学常数
覆盖 `<cmath>` 基础函数、C++17 特殊数学函数和 C++20 数学常数。**对应**：常用数学函数、特殊数学函数（C++17）、数学常数（C++20）。

### 20.2 随机数生成与分布
说明引擎与分布分离的设计，以及如何选择 `mt19937`、`random_device` 和标准分布。**对应**：伪随机数生成、引擎与分布。

### 20.3 `complex`、`valarray` 与基础线性代数/SIMD（C++26 前瞻）
介绍复数与数值数组的传统设施，并说明 C++26 基础线性代数算法和 SIMD 数据并行类型的方向。**对应**：`std::complex`、`std::valarray`、BLAS（C++26）、SIMD（C++26）。

## 第 21 章 时间库

### 21.1 `chrono` 基础：`duration`、`time_point`、`clock`
建立时间间隔与时间点的类型安全模型，说明三种时钟的语义差异。**对应**：`<chrono>`（C++11）、`duration`、`time_point`、`clock`。

### 21.2 日历与时区（C++20）
介绍 `year_month_day`、`weekday`、`zoned_time` 等日历组件和时区数据库接口。**对应**：日历（C++20）、时区（C++20）。


# 第四篇：Boost 准标准库精要

Boost 是 C++ 标准库的“预备役”和“补充层”。本篇按“生态定位 → 数据结构 → 文本处理 → 并发网络 → 通用工具 → 数值时间 → 测试工程”展开，帮助读者建立“标准库优先，Boost 补位”的选型判断。

## 第 22 章 Boost 的定位、生态与选型原则

### 22.1 Boost 与 C++ 标准库的共生关系
梳理 Boost 作为 C++ 标准化“试验场”的历史角色，以及同一设施在 Boost 与标准库之间的接口差异和取舍。**对应**：Boost.Optional / Variant / Any / Filesystem / Regex / Thread / Chrono / Coroutine2 等。

### 22.2 现代 C++ 项目中引入 Boost 的决策框架
给出“标准库能做就不引入 Boost”的基本纪律，以及必须引入 Boost 的三类情形。**对应**：Boost 整体选型原则、Boost 与 C++20 标准库的替代关系。

### 22.3 获取、构建与链接模式：header-only 与 compiled 库
说明纯头文件库与需要编译链接的库的区别，以及 `b2` / CMake `find_package(Boost)` 的基本用法。**对应**：Boost 构建系统、header-only vs compiled library。

## 第 23 章 Boost 容器与数据结构

### 23.1 `boost::multi_index`：多索引容器
介绍如何在一个数据集合上同时维护多种访问路径，替代“维护多个同步容器”的易错模式。**对应**：Boost.MultiIndex。

### 23.2 `boost::intrusive`：侵入式容器
说明侵入式容器如何将钩子嵌入用户类型、避免额外分配，以及其对对象生命周期的额外约束。**对应**：Boost.Intrusive。

### 23.3 `boost::container` 与 `boost::circular_buffer`
介绍 Boost.Container 的扩展容器和 `circular_buffer` 在流式缓冲和固定窗口场景中的定位。**对应**：Boost.Container、Boost.CircularBuffer。

### 23.4 `boost::bimap`、`boost::variant2` 与 `boost::any`
覆盖双向映射、支持 valueless-by-exception 策略的 variant 实现，以及 Boost 对词汇类型设施的持续演进。**对应**：Boost.Bimap、Boost.Variant2、Boost.Any。

## 第 24 章 Boost 字符串、文本与解析

### 24.1 `boost::algorithm` 与字符串算法扩展
介绍字符串裁剪、分割、连接、大小写转换等算法，以及它们如何补充标准库 `<algorithm>` 在字符串场景中的空白。**对应**：Boost.StringAlgo、Boost.Algorithm。

### 24.2 `boost::format`：类型安全的格式化
说明 `boost::format` 的占位符风格格式化与 `std::format` 的异同，以及在本项目尚未升级到 C++20 时的替代价值。**对应**：Boost.Format。

### 24.3 `boost::tokenizer` 与文本分词
覆盖将字符串或字符序列拆分为 token 的轻量设施。**对应**：Boost.Tokenizer。

### 24.4 `boost::spirit`：基于表达式模板的解析器生成
以“用接近 EBNF 的语法在 C++ 中直接编写解析器”为核心，展示 Spirit.Qi 的解析器组合子和属性合成机制。**对应**：Boost.Spirit（Spirit.Qi / Spirit.Karma）。

## 第 25 章 Boost 并发与网络

### 25.1 `boost::asio`：异步 I/O 与网络编程的统一模型
以 `io_context`、异步操作、完成处理器和 `strand` 为主线，说明 Asio 如何覆盖 TCP、UDP、定时器和串口，并介绍 `co_spawn` / `awaitable` 与 C++20 协程的衔接。**对应**：Boost.Asio（`io_context`、`strand`、`co_spawn`、`awaitable`）。

### 25.2 `boost::thread` 与同步设施的“前标准时代”价值
说明 `boost::thread`、`boost::mutex`、`boost::condition_variable` 在 C++11 之前的不可替代性，以及维护旧代码库时的持续用途。**对应**：Boost.Thread。

### 25.3 `boost::coroutine2`、`boost::fibers` 与 C++20 协程的对比
介绍 Boost 在栈式协程和用户态纤程方面的实现，并与 C++20 无栈协程在语义、调度模型和适用场景上做对比。**对应**：Boost.Coroutine2、Boost.Fiber。

## 第 26 章 Boost 通用工具与函数式设施

### 26.1 `boost::function`、`boost::bind` 与 `boost::signals2`
说明这些设施在 `std::function` / Lambda 普及之前的地位，以及 `signals2` 在线程安全信号/槽场景中仍然独特的作用。**对应**：Boost.Function、Boost.Bind、Boost.Signals2。

### 26.2 `boost::lexical_cast` 与类型转换
介绍字符串与数值/类型之间的简单转换，以及其与 `std::to_chars` / `from_chars`、`std::format` 的关系和性能边界。**对应**：Boost.LexicalCast。

### 26.3 `boost::property_map` 与 `boost::dynamic_bitset`
覆盖属性映射概念以及动态大小位集在标准 `bitset` 之外的补充。**对应**：Boost.PropertyMap、Boost.DynamicBitset。

## 第 27 章 Boost 数值、日期时间与文件系统

### 27.1 `boost::multiprecision`：多精度数值
介绍任意精度整数、有理数、浮点数和复数类型，以及定长与变长精度在性能和易用性上的取舍。**对应**：Boost.Multiprecision。

### 27.2 `boost::date_time` 与 `boost::chrono` 的补充定位
说明 `boost::gregorian` / `boost::posix_time` 在 C++20 日历/时区之前的日期处理能力，以及 `boost::chrono` 的过渡角色。**对应**：Boost.DateTime、Boost.Chrono。

### 27.3 `boost::filesystem`：C++17 标准文件系统库的前身
覆盖路径操作、目录遍历、文件状态查询和文件操作，并说明它与 `std::filesystem` 的接口对应关系和迁移策略。**对应**：Boost.Filesystem。

## 第 28 章 Boost 测试、序列化与工程设施

### 28.1 `boost::test`：单元测试框架
介绍测试用例、测试套件、断言宏和 header-only 用法，并说明其与 GoogleTest / Catch2 的定位差异。**对应**：Boost.Test。

### 28.2 `boost::serialization`：对象序列化
说明如何通过成员函数 `serialize` 或自由函数将对象持久化为文本、XML 或二进制归档，以及指针/引用追踪和版本控制机制。**对应**：Boost.Serialization。

### 28.3 `boost::graph`（BGL）：通用图算法与数据结构
介绍 BGL 的核心设计：图表示与算法通过概念解耦，算法可作用于多种图结构；并覆盖 visitor 机制和常用算法。**对应**：Boost.Graph（BGL、`adjacency_list`、visitor hooks）。


# 第五篇：工程实践、跨版本演进与附录

## 第 29 章 模块化工程与构建

### 29.1 C++20 模块在工程中的落地
讨论模块接口与实现分离、模块分区、全局模块片段、与头文件库互操作和构建系统依赖扫描。**对应**：模块（C++20）、模块分区、`import std`（C++23）。

### 29.2 CMake 对模块与包管理的进阶支持
介绍 CMake 的 C++ 模块依赖扫描、`install(EXPORT)` 与 `find_package` 的配合，以及 CMake 与 vcpkg 在持续集成中的协同。**对应**：CMake 模块支持、`install(EXPORT)`、vcpkg 二进制缓存（工程工具）。

## 第 30 章 测试、诊断与静态分析

### 30.1 单元测试框架选型与结构化测试
比较 GoogleTest、Catch2、Boost.Test 的定位，并说明如何用现代 C++ 特性编写可读测试。**对应**：测试框架（工程实践）、`static_assert` / `constexpr` 在测试中的使用。

### 30.2 Sanitizer、静态分析与 `std::stacktrace` 的配合
覆盖 AddressSanitizer、UndefinedBehaviorSanitizer、ThreadSanitizer 的使用场景，以及用 `stacktrace` 增强崩溃报告。**对应**：Sanitizer（工具）、`std::stacktrace`（C++23）。

## 第 31 章 跨标准版本演进地图

### 31.1 C++11/14/17/20/23/26 特性对照与迁移策略
以表格列出各版本关键语言特性与标准库设施，标注支持状态，并给出从 C++11 向 C++20/23 迁移的优先顺序。**对应**：全书语言特性与标准库设施综合对照。

## 附录

### A. 标准库头文件速查（按类别）
### B. 编译器与标准支持矩阵
### C. 核心概念与设施索引（按 cppreference 分类映射）
### D. CMake 命令参考

附录 D 按“命令类别 + 用途 + 关键参数 + 典型示例”组织，覆盖第 7 章及第 29 章涉及的 CMake 设施。

| 类别 | 命令 | 用途 | 关键参数 / 备注 |
|---|---|---|---|
| **项目初始化** | `cmake_minimum_required` | 声明最低 CMake 版本 | `VERSION <min>` |
| | `project` | 声明项目名称、版本、语言 | `VERSION`、`LANGUAGES`、`DESCRIPTION` |
| | `cmake_policy` | 管理策略设置 | `SET` / `PUSH` / `POP` |
| **目标创建** | `add_executable` | 创建可执行目标 | 目标名、源文件列表 |
| | `add_library` | 创建库目标 | `STATIC` / `SHARED` / `MODULE` / `INTERFACE` |
| | `add_custom_target` | 创建无输出的自定义目标 | 常用于代码生成、文档构建 |
| **目标属性** | `target_sources` | 为目标指定源文件 | 可与 `PUBLIC` / `PRIVATE` / `INTERFACE` 搭配 |
| | `target_include_directories` | 为目标指定头文件路径 | 支持 `BUILD_INTERFACE` / `INSTALL_INTERFACE` 生成器表达式 |
| | `target_compile_definitions` | 为目标指定预处理宏 | 替代 `add_definitions` |
| | `target_compile_options` | 为目标指定编译器选项 | 替代 `add_compile_options` |
| | `target_compile_features` | 为目标指定所需 C++ 标准特性 | `cxx_std_20` 等 |
| | `target_link_libraries` | 为目标指定链接库 | 支持 `PUBLIC` / `PRIVATE` / `INTERFACE` |
| | `target_link_options` | 为目标指定链接器选项 | 与 `target_compile_options` 对称 |
| | `target_precompile_headers` | 为目标指定预编译头 | 优化编译时间，谨慎使用 |
| **目标查询与设置** | `get_target_property` | 读取目标属性 | 可与任意属性名配合 |
| | `set_target_properties` | 设置目标属性 | 如 `OUTPUT_NAME`、`CXX_STANDARD` |
| **依赖与包管理** | `find_package` | 查找已安装的包 | `REQUIRED`、`COMPONENTS`、`CONFIG` / `MODULE` |
| | `FetchContent_MakeAvailable` | 下载并纳入构建树 | CMake 3.14+ |
| | `add_subdirectory` | 将子目录纳入构建 | 常用于 vendored 依赖 |
| | `install` | 定义安装规则 | `TARGETS`、`EXPORT`、`FILES`、`DIRECTORY` |
| | `install(EXPORT)` | 导出目标供 `find_package` 使用 | 与 `export()` 配合 |
| **预设与构建** | `CMakePresets.json` | 项目级配置/构建/测试预设 | `configurePresets` / `buildPresets` / `testPresets` |
| | `CMakeUserPresets.json` | 开发者本地覆盖预设 | 不纳入版本控制 |
| | `cmake --preset` | 按预设配置 | `cmake --preset <name>` |
| | `cmake --build --preset` | 按预设构建 | `cmake --build --preset <name>` |
| | `ctest --preset` | 按预设运行测试 | `ctest --preset <name>` |
| **变量与缓存** | `set` | 设置普通变量或缓存变量 | `CACHE`、`PARENT_SCOPE` |
| | `option` | 声明布尔选项 | `option(<name> "<desc>" <default>)` |
| | `CMAKE_TOOLCHAIN_FILE` | 指定工具链文件 | vcpkg 集成入口 |
| | `CMAKE_EXPORT_COMPILE_COMMANDS` | 导出编译命令数据库 | 供 clang-tidy / IDE 使用 |
| **条件与循环** | `if` / `elseif` / `else` / `endif` | 条件分支 | 支持 `DEFINED`、`TARGET`、`STREQUAL` 等条件 |
| | `foreach` / `endforeach` | 遍历列表 | `IN LISTS`、`RANGE` |
| | `while` / `endwhile` | 条件循环 | 谨慎使用，优先用 CMake 列表操作 |


## 写作建议

1. **工具切换点必须显式**：1.5 节点明“第一篇示例用 cxxsh 运行”，第 7 章开篇点明“从本章起告别 cxxsh，进入 CMake 工程化构建”，第 7.7 节的模板成为后续所有章节示例的基线。
2. **标准库章节以“选型”而非“API 罗列”驱动**：容器、智能指针、字符串视图、`expected`/`optional`/`variant`、同步原语、`iostream` vs `format`/`print`，每节都应给出决策依据和反例。
3. **C++20/23 是分水岭，但 C++11/14/17 的铺垫不可跳过**：概念、范围、协程、模块的威力依赖于 `auto`、移动语义、`constexpr`、`if constexpr` 和可变参数模板的前置理解。
4. **Boost 篇每条设施都回答“标准库能不能做”**：按“已被标准化 / 部分标准化 / 纯 Boost 独有”三分法标注，Asio 和 Spirit 各给一节，Serialization 和 BGL 以最小示例加选型建议为主。
5. **C++26 内容统一标注“前瞻”**：契约、`std::execution`、危险指针/RCU、BLAS/SIMD 以“方向 + 最小示例 + 支持状态”呈现，避免读者在生产中依赖。