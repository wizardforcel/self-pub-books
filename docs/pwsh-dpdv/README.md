# PowerShell 纵深

![](docs/img/pwsh-dpdv.png)

> **副标题**：从语言语法到 .NET 运行时与 Win32 系统编程
>
> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)
>
> 阶段：原创撰写（已完稿，共 33 章）

* [在线阅读](https://powershell-deep-dive.flygon.net)

## 简介

本书以《C# 本质论》的语言教学纵深、《CLR via C#》的运行时视角、《Windows 核心编程》的系统机制框架和《Windows 程序设计》的窗口/图形主线为四重参照系，系统讲解 PowerShell 的语法体系、对象管道与命令机制、.NET 类库设施、Win32 API 编程，以及脚本工程化与互操作。

全书遵循“语言 → 管道 → .NET → Win32 → 工程化”的递进逻辑，每一节按“知识点讲解 + 代码示例”组织，核心概念配 Mermaid 图表。

本书的三个特色：

1. **对照式教学**：全书始终以“PowerShell 与 C# 的异同”为线索，标出每一个“形似神异”的地方——`hidden` 不是 `private`、`-eq` 在集合上是过滤器、本机 SEH 异常会绕过 `try/catch`……
2. **实测校准**：书中关键语义均在本机 PowerShell 7.6 逐一验证，修正了多处常见以讹传讹的说法（如 `[Marshal]` 并非 PowerShell 类型加速器、`FillRect` 在 user32 而非 gdi32、`clean` 块的输出不会外显等）。
3. **工程判断**：不止讲“怎么用”，更讲“什么时候不该用”——四类必须下沉 Win32 的信号、七条 P/Invoke 安全清单、何时把核心逻辑下沉到 C# 类库。

## 适用读者与阅读环境

- **读者**：会基本电脑操作、没有编程基础也能从第 1 章读起；有 C#/运维背景的读者可把本书当作“PowerShell 与 .NET/Win32 的对照手册”。
- **环境**：建议 **PowerShell 7.4+**（书中示例在 PowerShell 7.6 实测）；第 19–28 章的 WinForms/GDI 内容需要 **Windows 10/11**。
- **运行示例**：将代码保存为 `.ps1` 运行；若遇到“禁止运行脚本”，先阅读第 31 章了解执行策略，或以当前用户范围执行 `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`。

## 目录

全部章节见 [SUMMARY.md](SUMMARY.md)，以下按部分列出。

### 第 1 部分 语言基础与类型系统

* [第 1 章 PowerShell 执行模型与程序结构](docs/ch1.md)
* [第 2 章 类型基础：CLR 类型在 PowerShell 中的“第一层皮肤”](docs/ch2.md)
* [第 3 章 变量、作用域与运算符](docs/ch3.md)
* [第 4 章 流程控制：与 C# 的“形似”与“神异”](docs/ch4.md)
* [第 5 章 函数与脚本块：PowerShell 的“方法”与“委托”](docs/ch5.md)
* [第 6 章 面向对象编程：类、对象、属性、继承、多态与接口](docs/ch6.md)
* [第 7 章 泛型、集合与 LINQ 风格查询](docs/ch7.md)
* [第 8 章 异常处理：从 C# 的 try/catch 到 PowerShell 的双错误通道](docs/ch8.md)

### 第 2 部分 对象管道与命令体系

* [第 9 章 命令发现与帮助系统](docs/ch9.md)
* [第 10 章 提供程序与驱动器：统一命名空间](docs/ch10.md)
* [第 11 章 管道深入：对象流、参数绑定与 $input](docs/ch11.md)
* [第 12 章 对象筛选、投影、排序与分组](docs/ch12.md)

### 第 3 部分 .NET 运行时设施

* [第 13 章 程序集、类型发现与反射](docs/ch13.md)
* [第 14 章 网络：从应用层 HTTP 到传输层 Socket](docs/ch14.md)
* [第 15 章 文本与正则：String 不可变性、StringBuilder 与 RegularExpressions](docs/ch15.md)
* [第 16 章 容器：从 ArrayList 的遗留教训到并发集合](docs/ch16.md)
* [第 17 章 线程与并发：Thread / ThreadPool / Task 与 PowerShell 运行空间](docs/ch17.md)
* [第 18 章 IO 深化：流、编码与大文件处理](docs/ch18.md)
* [第 19 章 图形界面：Windows Forms 与 PowerShell 的 STA 约束](docs/ch19.md)

### 第 4 部分 Win32 API 编程

* [第 20 章 基础必备知识：错误处理、字符字符串与内核对象](docs/ch20.md)
* [第 21 章 进程与作业](docs/ch21.md)
* [第 22 章 线程基础与调度](docs/ch22.md)
* [第 23 章 线程同步：用户模式与内核对象](docs/ch23.md)
* [第 24 章 内存管理：虚拟内存、堆与内存映射文件](docs/ch24.md)
* [第 25 章 动态链接库：基础、高级技术与注入](docs/ch25.md)
* [第 26 章 结构化异常处理与错误报告](docs/ch26.md)
* [第 27 章 窗口、消息与输入](docs/ch27.md)
* [第 28 章 图形与文本输出：GDI 与设备上下文](docs/ch28.md)
* [第 29 章 使用边界：PowerShell 中 Win32 调用的工程判断](docs/ch29.md)

### 第 5 部分 工程化与互操作

* [第 30 章 模块化：函数、脚本与模块](docs/ch30.md)
* [第 31 章 脚本安全与执行策略](docs/ch31.md)
* [第 32 章 P/Invoke 与 COM 互操作](docs/ch32.md)
* [第 33 章 与 C# 的协作边界：什么时候不该用 PowerShell](docs/ch33.md)

## 写作约定

- 代码块均标注语言（powershell / csharp / mermaid），可直接复制运行或渲染。
- 涉及系统修改的示例（注册表、执行策略、作业终止等）均在文中标注风险与清理方式。
- 每章末尾附“动手练习”，建议边读边跑。

## 贡献

发现错误或有改进建议，欢迎提 [Issue](https://github.com/wizardforcel/powershell-deep-dive/issues) 或 PR。校对类 PR（错别字、代码勘误、表述优化）尤其欢迎。

## 购买纸质书

![](dpcs/img/opendoccn_print_20261110.png)

## 赞助我

![](https://img-blog.csdnimg.cn/20200112005920729.png)
