# 附录

本附录提供四张速查表，供读者在写作、阅读和工程实践中快速查阅：

- **附录 A**：标准库头文件速查（按类别）
- **附录 B**：编译器与标准支持矩阵
- **附录 C**：核心概念与设施索引（按 cppreference 分类映射）
- **附录 D**：CMake 命令参考

---

## A. 标准库头文件速查（按类别）

| 类别 | 头文件 | 主要内容 | 标准 |
|---|---|---|---|
| 语言支持 | `<cstddef>` | `nullptr_t`、`ptrdiff_t`、`size_t` | C++11 |
| | `<cstdint>` | 固定宽度整数 `int32_t`、`uint64_t` 等 | C++11 |
| | `<climits>` / `<cfloat>` | C 风格整数/浮点极限宏 | C++11 继承 |
| | `<typeinfo>` | `std::type_info`、`std::bad_cast` | C++11 |
| | `<initializer_list>` | `std::initializer_list` | C++11 |
| 类型特征 | `<type_traits>` | `std::is_*`、`std::integral_constant`、`std::bool_constant` | C++11/14/17 |
| 概念库 | `<concepts>` | `std::same_as`、`std::integral`、`std::invocable` 等 | C++20 |
| 字符串 | `<string>` | `std::basic_string`、`std::string` | C++11 |
| | `<string_view>` | `std::basic_string_view`、`std::string_view` | C++17 |
| | `<cstring>` | C 风格字符串函数 | C++11 继承 |
| 字符分类 | `<cctype>` | `std::isdigit`、`std::isspace` 等 | C++11 继承 |
| | `<locale>` | 本地化设施 | C++11 |
| 格式化 | `<format>` | `std::format`、`std::formatter` | C++20 |
| | `<print>` | `std::print`、`std::println` | C++23 |
| 正则 | `<regex>` | `std::regex`、`regex_match`、`regex_search`、`regex_replace` | C++11 |
| 通用工具 | `<utility>` | `std::pair`、`std::move`、`std::forward`、`std::swap` | C++11 |
| | `<tuple>` | `std::tuple`、`std::apply`、`std::make_from_tuple` | C++11/17 |
| | `<optional>` | `std::optional` | C++17 |
| | `<variant>` | `std::variant`、`std::visit` | C++17 |
| | `<any>` | `std::any` | C++17 |
| | `<expected>` | `std::expected`、`std::unexpected` | C++23 |
| 函数对象 | `<functional>` | `std::hash`、`std::function`、`std::bind_front/back`、`std::ref`/`cref` | C++11/20/23 |
| 内存管理 | `<memory>` | 智能指针、`std::allocator`、未初始化内存算法 | C++11/14/17/20 |
| | `<memory_resource>` | `std::pmr::*` 多态分配器 | C++17 |
| | `<new>` | `operator new`/`delete`、定位 new | C++11 |
| 序列容器 | `<vector>` | `std::vector` | C++11 |
| | `<list>` | `std::list` | C++11 |
| | `<forward_list>` | `std::forward_list` | C++11 |
| | `<deque>` | `std::deque` | C++11 |
| | `<array>` | `std::array` | C++11 |
| 关联容器 | `<map>` | `std::map`/`std::multimap` | C++11 |
| | `<set>` | `std::set`/`std::multiset` | C++11 |
| | `<unordered_map>` | `std::unordered_map` 等 | C++11 |
| | `<unordered_set>` | `std::unordered_set` 等 | C++11 |
| 容器适配器 | `<stack>` | `std::stack` | C++11 |
| | `<queue>` | `std::queue`/`std::priority_queue` | C++11 |
| 非拥有视图 | `<span>` | `std::span` | C++20 |
| | `<mdspan>` | `std::mdspan` | C++23 |
| | `<flat_map>` | `std::flat_map`/`std::flat_set` | C++23 |
| 迭代器 | `<iterator>` | `std::iterator_traits`、`reverse_iterator`、插入迭代器 | C++11 |
| 范围库 | `<ranges>` | `std::ranges` 算法、`views::*` 适配器、`std::generator`（C++23） | C++20/23 |
| 算法库 | `<algorithm>` | 非修改/修改序列算法、排序、二分查找、堆 | C++11/14/17/20 |
| 数值算法 | `<numeric>` | `accumulate`、`reduce`、`transform_reduce`、`inner_product` | C++11/17 |
| 执行策略 | `<execution>` | `std::execution::seq`/`par`/`par_unseq` | C++17 |
| 常用数学 | `<cmath>` | `std::sqrt`/`sin`/`exp` 等、特殊数学函数（C++17） | C++11/17 |
| 数学常数 | `<numbers>` | `std::numbers::pi`、`e` 等 | C++20 |
| 复数 | `<complex>` | `std::complex` | C++11 |
| 数值数组 | `<valarray>` | `std::valarray` | C++11 |
| 随机数 | `<random>` | 引擎、`mt19937`、分布 | C++11 |
| 有理数 | `<ratio>` | `std::ratio`、`ratio_add` 等 | C++11 |
| 时间 | `<chrono>` | `duration`/`time_point`/`clock`、日历与时区（C++20） | C++11/20 |
| 流式 I/O | `<iostream>` | `std::cin/cout/cerr/clog` | C++11 |
| | `<fstream>` | `std::ifstream`/`ofstream`/`fstream` | C++11 |
| | `<sstream>` | `std::stringstream` | C++11 |
| | `<spanstream>` | `std::spanstream`（无分配流） | C++23 |
| | `<syncstream>` | `std::osyncstream`（并发输出同步） | C++20 |
| | `<iomanip>` | 格式化操纵符 `setw`/`setprecision` 等 | C++11 |
| 文件系统 | `<filesystem>` | `std::filesystem::path`、目录遍历、文件操作 | C++17 |
| 线程 | `<thread>` | `std::thread`/`std::jthread` | C++11/20 |
| 互斥量 | `<mutex>` | `std::mutex`/`recursive_mutex`/`lock_guard`/`unique_lock`/`scoped_lock` | C++11/17 |
| | `<shared_mutex>` | `std::shared_mutex`/`shared_lock` | C++17 |
| 原子 | `<atomic>` | `std::atomic`、`std::atomic_flag`、内存序、`wait/notify` | C++11/20 |
| 条件变量 | `<condition_variable>` | `std::condition_variable`/`condition_variable_any` | C++11 |
| 同步原语 | `<latch>` | `std::latch` | C++20 |
| | `<barrier>` | `std::barrier` | C++20 |
| | `<semaphore>` | `std::counting_semaphore` | C++20 |
| 协程 | `<coroutine>` | 协程支持库（协程 Traits 等） | C++20 |
| | `<generator>` | `std::generator` | C++23 |
| 断言 | `<cassert>` | `std::assert` | C++11 继承 |
| errno | `<cerrno>` | `errno` | C++11 继承 |
| 系统错误 | `<system_error>` | `std::error_code`/`std::error_condition` | C++11 |
| 标准异常 | `<stdexcept>` | `std::exception`/`std::runtime_error`/`std::logic_error` 等 | C++11 |
| 调用栈 | `<stacktrace>` | `std::stacktrace`/`std::stacktrace_entry` | C++23 |
| 特性测试宏 | `<version>` | `__cpp_lib_*` 库特性宏 | C++20 |

---

## B. 编译器与标准支持矩阵

下表反映了截至本书写作时（2026 年）主流编译器对主要 C++ 标准特性的支持大致情况。具体版本请以编译器官方发布说明为准，因为支持情况随小版本更新持续变化。实际可用性请以编译器提供的 `<version>` 头文件中的 `__cpp_*` / `__cpp_lib_*` 特性测试宏为准。

| 特性 | GCC | Clang | MSVC | 备注 |
|---|---|---|---|---|
| C++11 核心特性 | 4.8+ | 3.3+ | 19.00+ | 基本完备 |
| C++14 核心特性 | 5.0+ | 3.4+ | 19.00+ | 基本完备 |
| C++17 核心特性 | 7.0+ | 5.0+ | 19.10+ | 基本完备 |
| C++20 核心特性 | 10.0+ | 10.0+ | 19.28+ | 大部分完备，模块/协程部分需较新版本 |
| C++23 核心特性 | 14.0+ | 18.0+ | 19.37+ | 部分特性仍在实验 |
| `std::format` | 13.0+（libstdc++） | 14.0+（libc++） | 19.30+ | 需对应标准库版本 |
| `std::print` / `println` | 14.2+ | 17.0+ | 19.40+ | C++23，支持情况较新 |
| `std::stacktrace` | 14.0+（需 `-lstdc++exp`） | 16.0+ | 19.40+ | C++23，部分链接环境需额外库 |
| 模块（`import`/`export`） | 11.0+（`-fmodules-ts`） | 14.0+（`-fmodules-ts`） | 19.29+ | 构建系统支持参差不齐 |
| `std::ranges` | 11.0+ | 14.0+ | 19.28+ | 需较新标准库 |
| `std::expected` | 14.1+ | 17.0+ | 19.39+ | C++23 |
| `std::string_view` | 7.0+ | 5.0+ | 19.10+ | C++17 |
| `std::filesystem` | 8.0+ | 7.0+ | 19.14+ | C++17，早期需链接 `stdc++fs` |
| `std::jthread`/`stop_token` | 11.0+ | 14.0+ | 19.28+ | C++20 |
| `std::latch`/`barrier`/`semaphore` | 11.0+ | 14.0+ | 19.28+ | C++20 |
| `std::atomic<shared_ptr<T>>` | 11.0+ | 14.0+ | 19.28+ | C++20 |

**常见特性测试宏参考：**

| 宏 | 含义 | 示例值 |
|---|---|---|
| `__cplusplus` | 当前标准版本 | C++20 为 `202002L` |
| `__cpp_concepts` | 概念与约束 | `201907L`（C++20） |
| `__cpp_lib_ranges` | 范围库 | `201911L`（C++20） |
| `__cpp_lib_format` | `std::format` | `201907L`（C++20） |
| `__cpp_lib_print` | `std::print/println` | `202207L`（C++23） |
| `__cpp_lib_expected` | `std::expected` | `202202L`（C++23） |
| `__cpp_lib_stacktrace` | `std::stacktrace` | `202002L`（C++23） |
| `__cpp_coroutines` | 协程 | `201902L`（C++20） |
| `__cpp_modules` | 模块 | `201907L`（C++20） |

**探测示例：**

```cpp
#include <version>

#if defined(__cpp_lib_format) && __cpp_lib_format >= 201907L
#    include <format>
#endif
```

---

## C. 核心概念与设施索引（按 cppreference 分类映射）

| cppreference 分类 | 核心设施 | 本书章节 |
|---|---|---|
| 语言支持库 | `numeric_limits`、`type_info`、`type_index`、`initializer_list`、`cstdint`、`climits`/`cfloat` | 第 13 章 |
| 诊断库 | `assert`、`errno`、`error_code`/`error_condition`、`exception`、`stacktrace` | 第 14 章 |
| 内存管理库 | `unique_ptr`/`shared_ptr`/`weak_ptr`、`allocator`、`pmr`、`uninitialized_*` | 第 4、15 章 |
| 概念库 | `same_as`、`convertible_to`、`integral`、`floating_point`、`invocable`、`regular` | 第 5、16 章 |
| 元编程库 | `integral_constant`、`bool_constant`、`true_type`/`false_type`、`ratio` | 第 5、16 章 |
| 迭代器库 | `iterator_traits`、`reverse_iterator`、`move_iterator`、插入迭代器、哨兵 | 第 17 章 |
| 范围库 | `ranges` 算法、`views::filter/transform/take/drop/join`、`generator`、`ranges::to` | 第 18 章 |
| 算法库 | `sort`/`stable_sort`/`partial_sort`、`find`/`count`/`all_of`、`accumulate`/`reduce`、`execution` 策略 | 第 19 章 |
| 数值库 | `cmath` 函数、特殊数学函数、`numbers` 常数、`random`、`complex`、`valarray` | 第 20 章 |
| 时间库 | `chrono`、`duration`/`time_point`/`clock`、`year_month_day`、`zoned_time` | 第 21 章 |
| 通用工具库 | `pair`/`tuple`、`optional`/`variant`/`expected`、`bitset`、`hash`、`function` | 第 8 章 |
| 容器库 | `vector`/`list`/`forward_list`/`deque`/`array`、`map`/`set`、`unordered_map/unordered_set`、`span`/`mdspan`/`flat_map`/`flat_set` | 第 9 章 |
| 字符串文本正则 | `string`/`string_view`、`format`/`print`/`println`、`regex` | 第 10 章 |
| I/O 文件系统 | `iostream`、`fstream`、`sstream`、`spanstream`、`syncstream`、`filesystem` | 第 11 章 |
| 并发库 | `thread`/`jthread`、`stop_token`、`mutex`/`scoped_lock`、`condition_variable`、`atomic`、`latch`/`barrier`/`semaphore`、`generator` | 第 12 章 |
| Boost 准标准库 | `asio`、`multi_index`、`intrusive`、`circular_buffer`、`bimap`、`variant2`、`signals2`、`lexical_cast`、`property_map`、`dynamic_bitset`、`multiprecision`、`date_time`/`chrono`、`filesystem`、`test`、`serialization`、`graph`（BGL） | 第 22–28 章 |
| 工程工具 | CMake 现代目标化构建、`target_*` 命令、生成器表达式、`find_package`/`FetchContent`、`vcpkg` 清单、`CMakePresets.json` | 第 7、29 章 |

---

## D. CMake 命令参考

| 类别 | 命令 | 用途 | 关键参数 / 备注 |
|---|---|---|---|
| **项目初始化** | `cmake_minimum_required` | 声明最低 CMake 版本 | `VERSION <min>` |
| | `project` | 声明项目名称、版本、语言 | `VERSION`、`LANGUAGES`、`DESCRIPTION` |
| | `cmake_policy` | 管理策略设置 | `SET` / `PUSH` / `POP` |
| **目标创建** | `add_executable` | 创建可执行目标 | 目标名、源文件列表 |
| | `add_library` | 创建库目标 | `STATIC` / `SHARED` / `MODULE` / `INTERFACE` |
| | `add_custom_target` | 创建无输出的自定义目标 | 常用于代码生成、文档构建 |
| **目标属性** | `target_sources` | 为目标指定源文件 | 可与 `PUBLIC` / `PRIVATE` / `INTERFACE` 搭配 |
| | `target_include_directories` | 为目标指定头文件路径 | 支持 `BUILD_INTERFACE` / `INSTALL_INTERFACE` |
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
| **条件与循环** | `if` / `elseif` / `else` / `endif` | 条件分支 | 支持 `DEFINED`、`TARGET`、`STREQUAL` 等 |
| | `foreach` / `endforeach` | 遍历列表 | `IN LISTS`、`RANGE` |
| | `while` / `endwhile` | 条件循环 | 谨慎使用，优先用 CMake 列表操作 |

**常用生成器表达式：**

| 表达式 | 含义 |
|---|---|
| `$<CONFIG:Debug>` | 当前构建类型为 Debug 时为真 |
| `$<CXX_COMPILER_ID:GNU>` | C++ 编译器为 GCC 时为真 |
| `$<BUILD_INTERFACE:...>` | 仅在构建树中有效 |
| `$<INSTALL_INTERFACE:...>` | 仅在安装后有效 |
| `$<IF:condition,true_value,false_value>` | 条件表达式 |

---

附录结束。全书正文（第 1–31 章）与附录 A–D 已全部完成，可作为《现代 C++ 全景：语言特性、标准库与工程化实践》的完整正文基线。
