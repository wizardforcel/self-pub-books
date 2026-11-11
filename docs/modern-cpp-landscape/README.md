# 《现代 C++ 全景：语言特性、标准库与工程化实践》

![](img/

一本面向**零基础读者**的现代 C++ 中文教程。本书假设你只会电脑基本操作，没有学过任何编程语言，从最基本的声明与初始化讲起，一路覆盖 C++11 到 C++26 的语言特性、标准库常用设施、工程化构建（CMake + vcpkg）、Boost 准标准库，以及测试、诊断与跨版本迁移。

> 📖 本书正文由 31 章 + 附录 组成，所有章节均已编写完成，可直接阅读。

---

## 📚 书籍目录

### 第一篇：现代 C++ 基础与语言核心演进（第 1–7 章）

| 章 | 标题 | 内容概要 |
|---|---|---|
| [第 1 章](ch1.md) | 现代 C++ 的定位与工具链 | 标准演进、特性测试宏、翻译与链接、类型系统、值类别，以及用 cxxsh 像脚本一样运行第一个 Hello World |
| [第 2 章](ch2.md) | 声明、初始化与作用域 | `auto`/`decltype`、统一初始化、`{}` 列表初始化、结构化绑定、命名空间、`const`/`constexpr`/`consteval`/`constinit` |
| [第 3 章](ch3.md) | 函数、Lambda 与可调用对象 | 函数重载、默认参数、`constexpr` 函数、Lambda 表达式、`std::function`、`std::move_only_function`、完美转发 |
| [第 4 章](ch4.md) | 类、对象与资源管理 | 特殊成员函数与三/五法则、委托与继承构造、智能指针、RAII、`variant`/`any`/`union` |
| [第 5 章](ch5.md) | 模板、泛型与编译期抽象 | 函数/类/变量模板、CTAD、可变参数模板与折叠表达式、`if constexpr`、概念与约束、`type_traits`/`integer_sequence`/`ratio` |
| [第 6 章](ch6.md) | 模块、契约与错误处理模型 | C++20 模块、`std::error_code`、`std::expected`（C++23）、C++26 契约前瞻、`std::source_location` |
| [第 7 章](ch7.md) | CMake 与 vcpkg：现代 C++ 工程化基础 | 最小 CMake 工程、`target_*` 命令、生成器表达式、三种依赖获取方式、vcpkg 清单模式、`CMakePresets.json`、完整工程模板 |

> 🔧 **工具分水岭**：第 1–7 章的单文件示例用 `cxxsh`（c99sh 的 C++ 模式）像脚本一样直接运行，无需手动调用编译器；从第 7 章起，示例切换到 CMake + vcpkg 的工程化构建，第 7.7 节的模板成为后续章节的基线。

### 第二篇：标准库常用基础设施（第 8–12 章）

| 章 | 标题 | 内容概要 |
|---|---|---|
| [第 8 章](ch8.md) | 通用工具库 | `pair`/`tuple`、`apply`/`make_from_tuple`、`optional`/`variant`/`expected` 词汇类型、`bitset` 与位操作、`std::hash`、函数对象与引用包装器 |
| [第 9 章](ch9.md) | 容器库 | `vector`/`array`/`deque`/`list`/`forward_list`、`map`/`set` 与 `unordered_map/unordered_set`、`stack`/`queue`/`priority_queue`、`flat_map`/`flat_set`（C++23）、`span` 与 `mdspan` |
| [第 10 章](ch10.md) | 字符串、文本与正则库 | `std::string` 与 `std::string_view`、`std::format` 与 `std::print`（C++20/23）、字符分类与 Unicode 基础、正则表达式 |
| [第 11 章](ch11.md) | 输入/输出与文件系统库 | `iostream` 与格式化操纵符、`std::osyncstream`（并发输出）、`stringstream` 与 `spanstream`（C++23）、`<filesystem>` 文件系统库 |
| [第 12 章](ch12.md) | 并发支持库 | `thread`/`jthread` 与停止令牌、互斥量与条件变量、`latch`/`barrier`/`counting_semaphore`、原子与六种内存序、`atomic<shared_ptr>`、危险指针/RCU（C++26 前瞻）、协程与 `std::execution`（C++26 前瞻） |

### 第三篇：标准库底层支撑与计算专项（第 13–21 章）

| 章 | 标题 | 内容概要 |
|---|---|---|
| [第 13 章](ch13.md) | 语言支持库 | `numeric_limits`、固定宽度整数/浮点、`initializer_list`、`type_info`/`type_index`、特性测试宏 |
| [第 14 章](ch14.md) | 诊断库 | `assert`、`errno`、`std::error_code`、标准异常层次、`noexcept`、异常安全、`std::stacktrace`（C++23） |
| [第 15 章](ch15.md) | 内存管理库 | `new`/`delete` 与 `std::allocator`、多态分配器 `std::pmr`、`monotonic_buffer_resource`、未初始化内存算法 |
| [第 16 章](ch16.md) | 概念库与元编程库 | `same_as`/`convertible_to`/`integral`/`invocable` 等概念、`integral_constant`、`std::ratio` 编译期有理数 |
| [第 17 章](ch17.md) | 迭代器库 | 迭代器类别与 `iterator_traits`、`reverse_iterator`/`move_iterator`/插入迭代器、范围迭代器与哨兵（C++20） |
| [第 18 章](ch18.md) | 范围库 | `std::ranges` 算法、`filter`/`transform`/`take`/`drop`/`join` 视图、惰性求值、`std::generator`（C++23）、`ranges::to`（C++23） |
| [第 19 章](ch19.md) | 算法库 | 非修改/修改序列算法、排序与二分查找、堆算法、`accumulate`/`reduce`/`transform_reduce`、并行执行策略、C++20 受约束算法 |
| [第 20 章](ch20.md) | 数值库 | 常用与特殊数学函数、`std::numbers` 数学常数（C++20）、随机数生成与分布、`complex`、`valarray`、C++26 线性代数与 SIMD 前瞻 |
| [第 21 章](ch21.md) | 时间库 | `duration`/`time_point`/`clock`、`system_clock`/`steady_clock`、C++20 日历 `year_month_day` 与时区 `zoned_time` |

### 第四篇：Boost 准标准库精要（第 22–28 章）

| 章 | 标题 | 内容概要 |
|---|---|---|
| [第 22 章](ch22.md) | Boost 的定位、生态与选型原则 | Boost 与标准库的共生关系、引入 Boost 的决策框架、header-only 与 compiled 库的获取构建 |
| [第 23 章](ch23.md) | Boost 容器与数据结构 | `multi_index` 多索引容器、`intrusive` 侵入式容器、`container` 与 `circular_buffer` 环形缓冲、`bimap` 双向映射、`variant2` 与 `any` |
| [第 24 章](ch24.md) | Boost 字符串、文本与解析 | `boost::algorithm` 字符串算法、`boost::format` 类型安全格式化、`boost::tokenizer` 文本分词、`boost::spirit` 解析器生成 |
| [第 25 章](ch25.md) | Boost 并发与网络 | `boost::asio` 异步 I/O 与网络、`boost::thread` 同步设施、`boost::coroutine2` 与 `boost::fibers` 及 C++20 协程对比 |
| [第 26 章](ch26.md) | Boost 通用工具与函数式设施 | `boost::function`/`bind`、`boost::signals2` 信号槽、`boost::lexical_cast` 类型转换、`boost::property_map` 与 `boost::dynamic_bitset` |
| [第 27 章](ch27.md) | Boost 数值、日期时间与文件系统 | `boost::multiprecision` 多精度数值、`boost::date_time` 与 `boost::chrono`、`boost::filesystem`（`std::filesystem` 的前身） |
| [第 28 章](ch28.md) | Boost 测试、序列化与工程设施 | `boost::test` 单元测试框架、`boost::serialization` 对象序列化、`boost::graph`（BGL）图算法与数据结构 |

### 第五篇：工程实践、跨版本演进与附录（第 29–31 章 + 附录）

| 章/附录 | 标题 | 内容概要 |
|---|---|---|
| [第 29 章](ch29.md) | 模块化工程与构建 | C++20 模块在工程中的落地（接口/实现分离、模块分区、全局模块片段）、CMake 模块依赖扫描、`install(EXPORT)` 与 `find_package`、vcpkg 在 CI 中的协同 |
| [第 30 章](ch30.md) | 测试、诊断与静态分析 | GoogleTest/Catch2/Boost.Test 选型、结构化测试、`static_assert`/`constexpr` 在测试中的使用、AddressSanitizer/UBSan/TSan、`std::stacktrace` 增强崩溃报告 |
| [第 31 章](ch31.md) | 跨标准版本演进地图 | C++11/14/17/20/23/26 特性综合对照表、`__cplusplus` 取值表、从 C++11 向 C++20/23 迁移的优先顺序与策略 |
| [附录](appendix.md) | 附录 A–D | A. 标准库头文件速查（按类别）；B. 编译器与标准支持矩阵；C. 核心概念与设施索引（按 cppreference 分类映射）；D. CMake 命令参考 |

---

## ✍️ 写作特色

本书遵循以下写作要求，适合零基础读者自学：

1. **面向零基础**：只会电脑基本操作，没学过任何编程语言也能读懂。
2. **语言详细直白**：重要概念用大白话解释，允许适当“废话”，使行文更自然流畅。
3. **每节带代码示例**：每一节都附带可运行的 C++ 代码，逐行解释语言特性或标准库设施。
4. **核心概念配 Mermaid 图**：用流程图、状态图等可视化抽象概念（如值类别、构建流水线、并发原语等）。
5. **细纲驱动**：所有章节正文均依据 `asset/detail.md` 中的细纲编写。

---

## 🛠 工具链说明

本书贯穿两条工具主线：

| 阶段 | 工具 | 说明 |
|---|---|---|
| 第 1–7 章 | **c99sh / cxxsh** | 单文件 C++ 程序像脚本一样执行，无需接触编译器命令行。给文件加上 `#!/usr/bin/env cxxsh` 即可直接运行。 |
| 第 7 章起 | **CMake + vcpkg** | 真实项目的工程化构建。`target_*` 命令、生成器表达式、`find_package`/`FetchContent`、`vcpkg.json` 清单模式、`CMakePresets.json` 预设。 |

**快速上手第 1–7 章示例：**

```bash
# 安装 c99sh（以 Linux/macOS/WSL2 为例）
git clone https://github.com/RhysU/c99sh.git
cd c99sh && chmod +x c99sh
sudo cp c99sh /usr/local/bin/c99sh
sudo ln -s /usr/local/bin/c99sh /usr/local/bin/cxxsh

# 运行第 1 章示例
cxxsh ch1.md 中的代码片段   # 或将代码保存为 .cpp 加上 #!/usr/bin/env cxxsh 后直接执行
```

> Windows 原生命令提示符建议先安装 WSL2；本书示例默认在类 Unix 环境下操作。

---

## 🤝 如何贡献

欢迎提 Issue 或 Pull Request 指出错误、建议补充内容。在贡献前请注意：

- 新增示例代码请确保可编译通过，并遵循本书“每节带代码、核心概念配 Mermaid 图”的写作风格。
- 第 7 章之前的单文件示例请优先用 `cxxsh` 验证；第 7 章起请套用第 7.7 节的 CMake + vcpkg 工程模板。
- 涉及 C++23/C++26 特性的示例，请标注编译器要求或做特性宏降级，避免读者在旧环境下编译失败。

---

## 📄 许可证

本项目采用 [MIT 许可证](../LICENSE) 开源，欢迎自由阅读、传播与改编。

---

**开始阅读**：[第 1 章 现代 C++ 的定位与工具链](ch1.md)
