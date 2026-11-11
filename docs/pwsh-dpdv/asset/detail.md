# 《PowerShell 纵深》

## 副标题：从语言语法到 .NET 运行时与 Win32 系统编程

> **全书定位**：本书以《C# 本质论》的语言教学纵深、《CLR via C#》的运行时视角、《Windows 核心编程》的系统机制框架和《Windows 程序设计》的窗口/图形主线为四重参照系，系统讲解 PowerShell 的语法体系、对象管道与命令机制、.NET 类库设施、Win32 API 编程，以及脚本工程化与互操作。全书遵循“语言 → 管道 → .NET → Win32 → 工程化”的递进逻辑，每一节均按“简介—知识点—语法/类/函数”三段式展开。

## 全书结构概览

| 部分 | 主题 | 章号 | 核心目标 |
|---|---|---|---|
| 第1部分 | 语言基础与类型系统 | 第1–8章 | 建立“PowerShell 语法是 C# 语法的对象化变体”的认知 |
| 第2部分 | 对象管道与命令体系 | 第9–12章 | 掌握 PowerShell 作为壳的核心机制：对象流、参数绑定、命令发现 |
| 第3部分 | .NET 运行时设施 | 第13–19章 | 理解 PowerShell 如何暴露 CLR 类型系统、IO、网络、文本、容器、线程与图形界面 |
| 第4部分 | Win32 API 编程 | 第20–29章 | 在 PowerShell 中直接调用操作系统：系统机制、进程线程、内存、DLL、窗口与 GDI |
| 第5部分 | 工程化与互操作 | 第30–33章 | 从“能写脚本”到“能交付工具” |


# 第1部分 语言基础与类型系统：从 C# 语法到 PowerShell 表达式

> **部分定位**：PowerShell 的语法功能和关键字类似于 C#，但相似之下有一个根本转向：C# 中类型的“身份”由 CLR 元数据固定，PowerShell 中每个对象都被一层 `PSObject` 包裹，类型可以在运行时被“扩展”。第1–5章建立语法与对象管道的直觉，第6章集中处理 PowerShell 自有的类定义语法——它不是 C# 类系统的等价物，而是一个在 CLR 类型系统之上、但受 PowerShell 运行时约束的“轻量级 OOP 子集”。第7–8章回到泛型/异常这些 CLR 核心设施。

## 第1章 PowerShell 执行模型与程序结构

### 1.1 会话、运行空间与脚本生命周期
**简介**：本节说明一次 PowerShell 执行发生在什么容器里，以及 `.ps1` 脚本在“命令类型谱系”中的位置，为后续所有语法讨论建立运行时语境。
**知识点**：PowerShell 是构建在 CLR 之上的命令行 shell 与脚本语言，处理的是对象而非文本；四种命令类型——脚本、函数/方法、cmdlet、本机命令——的界限与调用关系；运行空间（Runspace）是 cmdlet 和脚本执行的实际容器，会话是运行空间之上的交互上下文。
**语法/类/函数**：`$Host`、`$PSCmdlet`、`[System.Management.Automation.Runspaces.Runspace]`、`[System.Management.Automation.PowerShell]::Create()`；`.ps1` 文件作为脚本单元的约定。

### 1.2 表达式模式与命令模式
**简介**：PowerShell 解析器在“表达式模式”（像 C# 一样求值）和“命令模式”（像 shell 一样以空格分隔参数）之间切换，这是初学者最早遭遇的“语法怪异”的根源。
**知识点**：两种模式的触发条件与切换规则；命令模式下参数是字符串字面量、表达式需要括号包裹；`--%` 停止解析标记的使用场景。
**语法/类/函数**：`Write-Output (1 + 2)` 与 `Write-Output 1 + 2` 的解析差异；`--%` 之后的原样传递语义；`$MyInvocation.Line` 查看当前调用行。

## 第2章 类型基础：CLR 类型在 PowerShell 中的“第一层皮肤”

### 2.1 基元类型、值类型与引用类型在 PowerShell 中的表现
**简介**：对标《CLR via C#》第5章“基元类型、引用类型和值类型”与《C# 本质论》第2–3章，但重点放在 PowerShell 中赋值、比较和装箱行为与 C# 的差异上。
**知识点**：PowerShell 字面量默认映射到的 .NET 类型（`1` → `Int32`，`1L` → `Int64`，`1.0` → `Double`，`"x"` → `String`）；值类型在管道和变量赋值中的装箱路径；`$null` 不是“空引用”而是“自动化空值”的特殊语义。
**语法/类/函数**：`[int]`、`[long]`、`[double]`、`[string]` 等类型加速器；`GetType()` 与 `[type]::GetType()`；`$a = 1; $a.GetType().IsValueType`。

### 2.2 PSObject 与扩展类型系统（ETS）
**简介**：PowerShell 中“对象”的运行时形态不是裸 CLR 对象，而是 `PSObject` 包装后的适配视图。不理解 ETS，就无法理解为什么 `Get-Process` 的输出“看起来有额外的属性”。
**知识点**：`PSObject` 作为所有管道对象的通用包装器；扩展类型系统（ETS）如何在不修改原始 .NET 类型的前提下添加属性、方法和类型名称；适配类型系统（ATS）对 COM、WMI 等异构对象系统的映射。
**语法/类/函数**：`[psobject]`、`$obj.PSObject.Properties`、`$obj.PSObject.Methods`、`Update-TypeData`、`Types.ps1xml` 的加载机制。

### 2.3 类型转换、强制转换与 `-as` / `[type]` 操作符
**简介**：PowerShell 的转换规则比 C# 更“宽容”也更“隐晦”：何时静默转换、何时抛错、何时得到 `$null`，是脚本健壮性的关键分水岭。
**知识点**：`[type]` 强制转换在失败时抛出异常，`-as` 在失败时返回 `$null`；语言级转换（LanguagePrimitives）的优先级；字符串到数值的解析规则与区域性影响。
**语法/类/函数**：`[int]"42"`、`"42" -as [int]`、`[System.Convert]::ChangeType()`、`[System.Management.Automation.LanguagePrimitives]::ConvertTo()`。

## 第3章 变量、作用域与运算符

### 3.1 变量声明、作用域链与自动化变量
**简介**：对标《C# 本质论》“变量与作用域”的讨论，但 PowerShell 没有块级 `{}` 作用域，取而代之的是“脚本/函数/全局/私有”的命名空间层级。
**知识点**：`$script:`、`$global:`、`$local:`、`$private:` 作用域限定符；变量提供程序（Variable provider）作为统一存储层；自动化变量（`$_`、`$PSItem`、`$args`、`$input`、`$PSCmdlet`、`$Error`）的注入时机。
**语法/类/函数**：`New-Variable`、`Set-Variable`、`Get-Variable`、`Remove-Variable`；`[System.Management.Automation.PSVariable]`；`$ExecutionContext.SessionState.PSVariable`。

### 3.2 运算符体系：比较、逻辑、位运算与空合并
**简介**：PowerShell 的比较运算符以 `-eq` 系列命名，但在集合语义下，它们的行为从“标量比较”扩展为“过滤”——这是 C# 程序员最容易误判的地方。
**知识点**：`-eq` / `-ne` / `-gt` 等在左操作数为集合时返回匹配元素而非布尔值；`-contains` / `-in` 与 `-eq` 的集合语义差异；`-and` / `-or` 的短路行为与真值转换规则；`-band` / `-bor` / `-bxor` / `-shl` / `-shr`；空合并 `??` 与空条件 `?.` 在 PowerShell 7+ 中的支持。
**语法/类/函数**：`1..10 -gt 5` 的结果集合；`$null ?? "default"`；`[System.Management.Automation.LanguagePrimitives]::IsTrue()`。

### 3.3 格式化运算符 `-f` 与字符串插值
**简介**：字符串构建是脚本的主要“输出面”，PowerShell 同时提供了 C# 风格的 `-f` 格式化、双引号内插值和 here-string 三种机制，各有适用边界。
**知识点**：`-f` 对标 `String.Format` 的复合格式化；双引号内 `$var` 与 `$($expression)` 的展开时机；单引号字面量与 here-string（`@""@`、`@''@`）的转义规则差异。
**语法/类/函数**：`"{0} has {1} items" -f $name, $count`；`[string]::Format()`；`[System.String]::IsNullOrEmpty()` 与 `[string]::IsNullOrWhiteSpace()`。

## 第4章 流程控制：与 C# 的“形似”与“神异”

### 4.1 条件分支：`if` / `elseif` / `else` 与 `switch`
**简介**：`if` 的语法与 C# 几乎同形，但 `switch` 在 PowerShell 中是一个“多模式匹配器”，远超 C# 的常量分支语义。
**知识点**：`if` 的真值转换规则（非空字符串、非零数字、非 `$null` 为真）；`switch` 的默认通配符/正则/精确匹配模式（`-Wildcard`、`-Regex`、`-Exact`）；`switch` 对集合的逐项迭代语义；`break` / `continue` 在 `switch` 中的行为。
**语法/类/函数**：`switch -Regex ($line) { '^\d+' { ... } }`；`switch ($x) { { $_ -gt 10 } { ... } }` 的脚本块 case。

### 4.2 循环：`for`、`foreach`、`while`、`do-while` 与 `do-until`
**简介**：PowerShell 的循环语法与 C# 高度相似，但 `foreach` 语句与 `ForEach-Object` cmdlet 是两套不同的机制，混用是常见性能陷阱。
**知识点**：`foreach` 语句在内存中枚举集合，`ForEach-Object` 在管道中流式处理；`while` 与 `do-until` 的条件求值时机差异；循环中的 `break` / `continue` 作用范围。
**语法/类/函数**：`foreach ($x in 1..10) { }`、`1..10 | ForEach-Object { $_ * 2 }`、`[System.Collections.IEnumerator]` 的 `MoveNext()` / `Current`。

### 4.3 管道作为“流控制原语”：`ForEach-Object` / `Where-Object` / `Select-Object`
**简介**：在 PowerShell 中，管道不仅是数据通道，也是延迟执行的流控制手段。对标《C# 本质论》的 LINQ 章节，可以把这三个 cmdlet 理解为“命令式语法的 `Where` / `Select` / `ForEach`”。
**知识点**：管道逐对象流式处理模型与 LINQ 延迟执行的类比；`Where-Object` 的简化语法（`-Property` / `-Value`）与脚本块语法；`Select-Object` 的属性投影与 `-First` / `-Last` / `-Skip` 的提前终止语义。
**语法/类/函数**：`Where-Object { $_.Status -eq 'Running' }`、`Select-Object Name, CPU`、`ForEach-Object -Parallel`（PowerShell 7+）。

## 第5章 函数与脚本块：PowerShell 的“方法”与“委托”

### 5.1 基本函数、参数传递与 `$args`
**简介**：对标《C# 本质论》的“方法”章和《CLR via C#》第8–9章“方法/参数”，但 PowerShell 函数的参数绑定是运行时、声明式且高度可扩展的。
**知识点**：`function` 关键字与 `param` 块；位置参数、命名参数与剩余参数；`$args` 数组与 `param` 块的共存与优先级；简单函数与高级函数的界限。
**语法/类/函数**：`function Get-Thing { param($Name, [int]$Count = 1) ... }`；`$args.Count`；`$PSBoundParameters`。

### 5.2 高级函数：`[CmdletBinding()]`、参数验证与通用参数
**简介**：高级函数是“用户编写的 cmdlet”，它获得了与编译 cmdlet 相同的参数绑定、`-WhatIf` / `-Confirm`、`-Verbose` / `-Debug` 等通用设施，是脚本走向工具化的第一道门。
**知识点**：`[CmdletBinding()]` 的注入效果；`[Parameter()]` 的 `Mandatory`、`Position`、`ValueFromPipeline`、`ValueFromPipelineByPropertyName`；验证属性 `[ValidateSet()]`、`[ValidateRange()]`、`[ValidatePattern()]`、`[ValidateScript()]`；`SupportsShouldProcess` 与 `$PSCmdlet.ShouldProcess()`。
**语法/类/函数**：`[CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]`；`[ValidateNotNullOrEmpty()]`；`$PSCmdlet.WriteVerbose()`。

### 5.3 脚本块作为一等值与延迟绑定
**简介**：`{ ... }` 在 PowerShell 中不只是一个代码块，而是可传递、可远程执行、可作为参数“延迟求值”的 `ScriptBlock` 对象——它同时承担了 C# 中委托、Lambda 和部分反射调用的角色。
**知识点**：`ScriptBlock` 的字面量与 `[scriptblock]::Create()` 的动态创建；延迟绑定脚本块（delay-bind scriptblock）在管道参数绑定中的隐式触发；`.Invoke()`、`&` 调用运算符与 `.` 点源调用的作用域差异。
**语法/类/函数**：`$sb = { param($x) $x * 2 }; & $sb 3`；`Get-ChildItem | Select-Object @{Name='Size'; Expression={ $_.Length }}`（哈希表形式的计算属性本质上是脚本块）。

## 第6章 面向对象编程：类、对象、属性、继承、多态与接口

> **章内定位**：对标《C# 本质论》第5–9章“类与面向对象编程”和《CLR via C#》第6章“类型和成员基础”。但本章的核心教学任务不是“教 OOP”——两本参照著作已经完成——而是教 **PowerShell 的 OOP 与 C# 的 OOP 之间的系统性差异**：`hidden` 不是 `private`，属性不是字段但也没有自动实现，重写是隐式的因而没有编译期契约检查，接口实现一旦遗漏就是运行时解析错误而非编译错误。不理解这些差异，从 C# 转来的开发者会持续写出“看起来对但行为不对”的 PowerShell 类。

### 6.1 类定义与对象实例化

#### 6.1.1 `class` 关键字、类型定义与 `[ClassName]::new()`
**简介**：对标《C# 本质论》第5章“类与对象”和《CLR via C#》第6章“类型基础”。PowerShell 5.0 起，`class` 关键字使用与 C# 高度相似的块语法定义自定义类型。类定义是创建对象实例的“蓝图”，类名即类型名，通过 `[ClassName]::new()` 或 `New-Object -TypeName` 实例化。与 C# 的关键差异在于：PowerShell 类定义是**会话级类型**，一旦定义便不可卸载或重定义；在模块中定义的类默认不导出，需要 `using module` 显式导入。
**知识点**：`class <name> { ... }` 的块语法；`[ClassName]::new()` 实例化时类名必须带方括号（方括号是 PowerShell 的类型标记）；`New-Object -TypeName` 的等价路径；类定义的会话生命周期——同一会话中重复定义同名类会报错；类与 `PSCustomObject` 的根本区别：前者是编译期类型，后者是运行时成员包。
**语法/类/函数**：`class Device { [string]$Brand }`；`[Device]::new()`；`New-Object -TypeName Device`；`[Device]@{ Brand = 'x' }` 的转换式构造（要求默认构造函数存在）。

#### 6.1.2 构造函数、`$this` 与构造函数重载
**简介**：对标《CLR via C#》“实例构造器”和《C# 本质论》“构造器”章。PowerShell 类支持无参构造、带参构造和静态构造，构造函数名必须与类名相同。`$this` 是实例成员的访问前缀，等价于 C# 的 `this`。与 C# 的关键差异：**构造函数不被继承**——每个派生类必须定义自己的构造函数；没有 `private` 构造函数，`hidden` 只能隐藏构造函数使其不出现在 IntelliSense 中，但无法阻止调用。
**知识点**：`ClassName() { ... }` 无参构造；`ClassName([type]$arg) { $this.Prop = $arg }` 带参构造；构造函数重载按参数类型签名选择；静态构造函数 `static ClassName() { ... }` 初始化类级状态；派生类构造函数通过 `: base(<args>)` 显式调用基类构造；`$this` 在实例方法/属性/构造函数中的通用性。
**语法/类/函数**：`class X { X() { } X([int]$a) { $this.A = $a } }`；`class D : B { D() : base(1) { } }`；`static X() { ... }`；`$this.GetType().FullName`。

### 6.2 属性、方法与成员控制

#### 6.2.1 类型化属性、验证特性与“属性即字段”的语义
**简介**：对标《C# 本质论》“属性”章和《CLR via C#》“属性”节。PowerShell 类中的数据成员在语法上写作 `[type]$Name`，但语义上是**属性而非字段**——即使没有显式的 getter/setter 块。属性可以附加验证特性（`[ValidateRange()]`、`[ValidateSet()]`、`[ValidateNotNullOrEmpty()]` 等）在赋值时执行约束。与 C# 的关键缺失：**没有自动属性语法**（`public int X { get; set; }`），也没有显式的 `get { }` / `set { }` 块；如果需要只读语义或赋值时副作用，只能通过 `hidden` 后备成员 + 方法或 `PSScriptProperty` 变通。
**知识点**：`[type]$Prop` 的声明语法；`[ValidateSet('A','B')]` / `[ValidateRange(1,100)]` 在属性赋值时触发验证；`hidden [type]$Prop` 使属性不出现在默认 `Get-Member` 和 IntelliSense 中（但 `-Force` 可见）；`static [type]$Prop` 的类级共享语义；属性没有“后备字段”与“访问器”的分离——属性赋值就是直接写入存储。
**语法/类/函数**：`[ValidateRange(1,100)] [int]$Count`；`hidden [string]$InternalState`；`static [int]$InstanceCount = 0`；`Get-Member -Force` 查看隐藏成员。

#### 6.2.2 实例方法、静态方法与 `hidden` 方法
**简介**：对标《CLR via C#》第8–9章“方法/参数”。PowerShell 类方法默认是实例方法，`static` 修饰符将其提升为类级方法。方法可以重载——同名方法通过参数类型区分。与 C# 的关键差异：**没有 `public` / `private` / `protected` / `internal` 访问修饰符**；`hidden` 仅隐藏成员的发现性（IntelliSense、`Get-Member` 默认输出），**不改变可访问性**——隐藏方法仍然可以从外部调用。这意味着 PowerShell 类无法实现真正的封装边界。
**知识点**：`[returnType] MethodName([type]$param) { ... }` 实例方法；`static [returnType] MethodName(...)` 静态方法；方法重载按参数类型签名解析；`hidden [type] Method(...)` 隐藏方法；`$this` 在实例方法中访问当前实例；静态方法中不能用 `$this`，必须用 `[ClassName]::StaticMember`。
**语法/类/函数**：`[string] ToString() { return $this.Name }`；`static [int] Add([int]$a, [int]$b) { return $a + $b }`；`hidden [void] InternalHelper() { ... }`；`[ClassName]::Add(1,2)`。

### 6.3 继承、方法重写与多态的“隐式”形态

#### 6.3.1 单一继承、基类构造调用与成员继承
**简介**：对标《C# 本质论》“继承”章和《CLR via C#》“类型继承”节。PowerShell 类支持**单一继承**（一个类只能有一个基类），但继承是可传递的。派生类继承基类的实例属性、实例方法和静态成员，但**不继承构造函数**（静态构造和实例构造都不继承）。派生类使用 `: Base` 语法声明继承关系，基类必须位于冒号后的最左侧。
**知识点**：`class Derived : Base { ... }` 语法；单一继承限制——不能 `class D : A, B`（`B` 只能作为接口）；继承的传递性（D → C → B → A）；派生类自动获得基类的属性（包括 `hidden` 属性）和方法（包括重载）；派生类构造函数必须通过 `: base(...)` 显式调用基类构造。
**语法/类/函数**：`class Album : PublishedWork { [string[]]$Genres }`；`class D : B { D() : base('arg') { } }`；`[DerivedClass]::StaticMethod()` 对继承的静态方法的调用。

#### 6.3.2 隐式重写：重新定义即覆盖，没有 `virtual` / `override`
**简介**：这是 **PowerShell 类与 C# 类差异最剧烈的地方**。C# 使用 `virtual` / `abstract` / `override` 关键字建立编译期契约：基类显式声明“此方法可被重写”，派生类显式声明“此方法重写了基类版本”，编译器负责检查签名匹配。PowerShell 类**没有这些关键字**；方法重写通过在派生类中**重新定义同名同参数类型签名的方法**隐式发生。这导致两个后果：第一，基类无法“禁止”某个方法被重写；第二，派生类的重写是“完全替换”——重载的方法在派生类中重新定义后，基类的对应重载**不再可用**，且**不能通过 `: base(...)` 调用基类的方法实现**（构造函数之外）。
**知识点**：隐式重写的语法——派生类中定义与基类“同名 + 参数类型序列相同”的方法即触发覆盖；重写的签名匹配规则：参数类型必须相同，返回类型可以不同；重写完全替换基类重载——不能部分保留；**没有 `base.Method()` 调用**，若需要在派生类方法中复用基类逻辑，只能将基类逻辑提取到 `hidden` 辅助方法中并从两个类中调用；静态方法同样可以被“重写”（在派生类中重新定义），但通过基类名调用时仍然执行基类版本，通过派生类名调用时执行派生类版本。
**语法/类/函数**：`class Derived : Base { [string] ToString() { return 'D' } }` 覆盖 `Base` 的 `ToString()`；`class B { static [datetime] Now() { Get-Date } }` / `class D : B { static [datetime] Now() { [B]::Now() } }`（通过基类名复用）；`$obj.GetType().GetMethod('M').DeclaringType` 确认实际执行的方法所属类型。

### 6.4 接口：契约的声明与实现

#### 6.4.1 接口继承语法与运行时契约检查
**简介**：对标《C# 本质论》“接口”章和《CLR via C#》“接口”节。PowerShell 类可以通过 `class C : System.IFormattable, System.IComparable { ... }` 的语法实现一个或多个 .NET 接口。与 C# 的关键差异：**接口实现没有编译期检查**——如果类声明了接口但没有实现全部成员，错误在**类解析时**才抛出，而不是在编写时由编辑器/编译器捕获。此外，PowerShell 不支持在 PowerShell 脚本中**定义**新接口；只能实现已有的 .NET 接口。
**知识点**：`: Interface1, Interface2` 的接口列表语法（基类之后，逗号分隔）；实现接口必须提供所有成员的匹配签名；`System.IFormattable` 要求 `ToString([string], [System.IFormatProvider])` 签名；`System.IComparable` 使 `-gt` / `-lt` 等运算符可用（否则运算符只做引用相等检查）；`System.Collections.IEqualityComparer` 等集合相关接口在 PowerShell 类中的实现。
**语法/类/函数**：`class Temperature : System.IFormattable { [string] ToString([string]$Format, [System.IFormatProvider]$FormatProvider) { ... } }`；`class A : System.Collections.IEqualityComparer { [bool] Equals($x, $y) { ... } [int] GetHashCode($obj) { ... } }`；`$obj -is [System.IFormattable]` 的运行时接口检查。

#### 6.4.2 接口在 PowerShell 中的实际用途：运算符、格式化与集合协议
**简介**：PowerShell 中实现接口的首要动机不是“设计模式”，而是**让自定义类参与 PowerShell 语言的既有协议**：`-eq` 的相等语义、`-gt` 的比较语义、字符串插值时的格式化语义、以及作为泛型集合元素时的相等性/排序行为。对标《CLR via C#》“接口”章的核心观点：接口是类型与运行时设施之间的契约。
**知识点**：不实现 `IComparable` 时 `-gt` 的行为（引用比较或抛出 `InvalidOperation`）；实现 `IFormattable` 后 `"{0:C}" -f $obj` 和 `$obj.ToString('C')` 的可用性；实现 `IEqualityComparer` 对 `HashSet<T>` 和 `Dictionary<TKey,TValue>` 的相等性判断的影响；**PowerShell 边界**：接口实现是 PowerShell 类获得“第一等 .NET 类型公民”身份的主要途径，没有接口的 PowerShell 类在管道和泛型集合中只能依赖 `PSObject` 的 ETS 适配。
**语法/类/函数**：`$obj -is [System.IComparable]`；`[System.IFormattable]` 的 `ToString` 签名；`[System.Collections.Generic.HashSet[MyClass]]::new()` 配合 `IEqualityComparer`。

### 6.5 PowerShell OOP 的边界与替代方案

#### 6.5.1 何时用 `class`，何时用 `PSCustomObject` / 哈希表
**简介**：PowerShell 提供了一条从“最轻”到“最重”的对象构造光谱：`@{}` 哈希表 → `[pscustomobject]` → `class`。对标《C# 本质论》“匿名类型 vs 具名类型”的取舍问题。`PSCustomObject` 适合**一次性数据载体**和管道输出塑形；`class` 适合**需要类型身份、方法行为、验证约束和跨多次调用的可复用契约**的场景。但 PowerShell 类缺乏封装和编译期检查，因此它不适合承担 C# 中“大型领域模型”的角色。
**知识点**：`[pscustomobject]@{ ... }` 的成员是 `NoteProperty`，无类型约束、无方法、无验证；`class` 的成员有静态类型、可附加验证特性、可定义方法；`PSCustomObject` 无法作为 `Add-Type` / 泛型类型参数的类型实参；`class` 在模块导出中的额外复杂性（`using module`）；**判断信号**：如果对象只需要“被读取和管道传递”，用 `PSCustomObject`；如果需要“在多个函数之间作为有契约的参数类型”，用 `class`；如果需要“真正的封装和继承契约”，应该下沉到 C#（参见第33章）。
**语法/类/函数**：`[pscustomobject]@{ Name='x'; Value=1 }`；`class Config { [ValidateSet('A','B')] [string]$Mode }`；`$obj -is [Config]` 的类型身份检查。

#### 6.5.2 PowerShell 类的运行时约束：`NoRunspaceAffinity` 与并行脚本块
**简介**：PowerShell 类默认与创建它的运行空间关联；在 `ForEach-Object -Parallel` 的脚本块中使用 PowerShell 类是不安全的，方法调用会被封送回创建运行空间，可能导致状态损坏或死锁。PowerShell 7.4 引入了 `NoRunspaceAffinity` 类属性来解除这一关联，使类实例可以在并行脚本块中使用。对标《CLR via C#》的“线程关联性”问题：这是 PowerShell 运行时约束在 OOP 层面的直接体现。
**知识点**：默认 `class` 的运行空间关联性；`ForEach-Object -Parallel` 中调用 PowerShell 类方法的封送风险；`[NoRunspaceAffinity()]` 属性（PowerShell 7.4+）的声明方式与效果——实例和静态方法调用使用当前线程的运行空间和会话状态；即使使用 `NoRunspaceAffinity`，类实例的**可变共享状态**仍然需要并发集合或同步原语（参见第16–17章）。
**语法/类/函数**：`[NoRunspaceAffinity()] class MyClass { ... }`；`ForEach-Object -Parallel { [MyClass]::new() }` 在有/无该属性时的行为差异。

## 第7章 泛型、集合与 LINQ 风格查询

### 7.1 .NET 泛型集合在 PowerShell 中的使用
**简介**：PowerShell 本身没有 C# 那样的泛型类型参数语法，但它可以完整地实例化和操作 .NET 泛型集合，并且通过类型推断在多数场景中隐藏了复杂性。对标《C# 本质论》第12章“泛型”和《CLR via C#》第12章。
**知识点**：`List<T>`、`Dictionary<TKey,TValue>`、`HashSet<T>` 在 PowerShell 中的构造与使用；`[System.Collections.Generic.List[int]]::new()` 与 `New-Object` 的等价性；泛型类型在管道中的展开行为；`ArrayList`（非泛型）与 `List<T>` 的选型边界；PowerShell 不支持在脚本中**定义**泛型类型。
**语法/类/函数**：`[System.Collections.Generic.List[string]]::new()`、`$list.Add()`、`$dict[$key]`、`[System.Linq.Enumerable]::Where()` 在 PowerShell 中的可调用性。

### 7.2 哈希表、有序字典与自定义对象
**简介**：哈希表是 PowerShell 脚本中最高频的“轻量数据结构”，而 `PSCustomObject` 是“无类定义的记录类型”，两者共同覆盖了 C# 中匿名类型和简单 DTO 的用途。
**知识点**：`@{}` 哈希表与 `[ordered]@{}` 有序字典；`[pscustomobject]` 的类型加速器语法；`Select-Object` 的 `-Property` 计算属性返回 `PSCustomObject`；`Add-Member` 动态添加成员。
**语法/类/函数**：`[pscustomobject]@{ Name = 'x'; Value = 1 }`；`[System.Collections.Specialized.OrderedDictionary]`；`Add-Member -MemberType NoteProperty`。

### 7.3 LINQ 风格操作：`Where-Object` / `Select-Object` / `Sort-Object` / `Group-Object` / `Measure-Object`
**简介**：PowerShell 的“标准查询操作符”以 cmdlet 形式存在，与 C# LINQ 的方法语法一一对应，但执行模型是流式管道而非表达式树编译。
**知识点**：过滤、投影、排序、分组、聚合五类操作的 cmdlet 映射；`Sort-Object` 的多键排序与 `-Descending`；`Group-Object` 的分组键与计数；`Measure-Object` 的 `-Sum` / `-Average` / `-Maximum` / `-Minimum`。
**语法/类/函数**：`Sort-Object -Property Length -Descending`、`Group-Object Status`、`Measure-Object -Property WorkingSet -Sum`。

## 第8章 异常处理：从 C# 的 `try/catch` 到 PowerShell 的双错误通道

### 8.1 终止性错误与非终止性错误
**简介**：对标《CLR via C#》第20章“异常和状态管理”与《C# 本质论》第10章，但 PowerShell 的异常模型多了一条“非终止性错误”通道——它不打断管道，却会写入 `$Error` 流。
**知识点**：终止性错误（terminating error）触发 `try/catch`；非终止性错误（non-terminating error）默认只写入错误流；`$ErrorActionPreference` 与 `-ErrorAction` 对两条通道的收束作用；`$Error` 自动变量作为环形缓冲区。
**语法/类/函数**：`$ErrorActionPreference = 'Stop'`；`-ErrorAction SilentlyContinue`；`$Error[0]`；`[System.Management.Automation.ErrorRecord]`。

### 8.2 `try` / `catch` / `finally` 与 `trap`
**简介**：`try/catch/finally` 的语法与 C# 同形，但 `catch` 块中的 `$_` 是 `ErrorRecord` 而非裸 `Exception`；`trap` 是 PowerShell 独有的语句级错误拦截机制。第6章定义的 PowerShell 类可以作为自定义异常的类型基础（`class MyException : System.Exception { ... }`），但 catch 的类型过滤仍然依赖 .NET 异常类型层次，而非 PowerShell 类的层次。
**知识点**：`catch` 的类型过滤（`catch [System.IO.FileNotFoundException]`）；`$_.Exception` 获取内部异常；`$_.Exception.InnerException` 与 `ErrorRecord` 的嵌套关系；`finally` 在成功、失败和 `Ctrl+C` 下的执行保证；`trap` 的作用域与 `continue` / `break` 行为。
**语法/类/函数**：`try { } catch [System.Exception] { $_.Exception.Message } finally { }`；`trap { ...; continue }`；`$_.ScriptStackTrace`。


# 第2部分 对象管道与命令体系：PowerShell 作为“壳”的核心机制

> **部分定位**：C# 程序的控制流由语言关键字主导；PowerShell 的控制流在很大程度上由管道和命令绑定规则主导。本部分不重复“常用命令清单”，而是把常用命令放进“命令发现 → 参数绑定 → 对象流 → 格式化输出”的机制链条中讲授。这是全书中唯一不直接对标 C# 著作的部分，但它是 PowerShell 成为 PowerShell 的原因。

## 第9章 命令发现与帮助系统

### 9.1 Cmdlet 命名约定与 `Get-Command`
**简介**：PowerShell 的命令遵循“动词-名词”单数形式，这一约定本身就是一套可发现、可预测的 API 设计哲学。
**知识点**：`Get-Verb` 返回的批准动词列表；动词的名词化语义（`Get` 检索、`Set` 配置、`New` 创建、`Remove` 删除、`Invoke` 执行）；`Get-Command` 按名称、模块、命令类型过滤；别名、函数、外部脚本与 cmdlet 的区分。
**语法/类/函数**：`Get-Command -Verb Get -Noun *Service`、`Get-Command -Module Microsoft.PowerShell.Management`、`Get-Alias`。

### 9.2 帮助系统与参数发现
**简介**：`Get-Help` 不只是文档入口，它暴露的参数元数据是编写健壮脚本时“逆向确认绑定规则”的工具。
**知识点**：`Get-Help` 的 `-Examples`、`-Parameter`、`-Full` 视图；`about_*` 主题的离线与在线获取；`Get-Member` 作为“对象结构发现器”与帮助系统的互补关系。
**语法/类/函数**：`Get-Help Get-Process -Parameter Id`、`Get-Help about_Pipeline_Chain`、`Get-Member -MemberType Property`。

## 第10章 提供程序与驱动器：统一命名空间

### 10.1 PSProvider 与 PSDrive 模型
**简介**：文件系统、注册表、证书存储、环境变量在 PowerShell 中共享同一套 `Get-ChildItem` / `Get-Item` / `Set-Item` 命令，这是“用统一 Shell 方式访问不同资源视图”的核心抽象。
**知识点**：内置提供程序（`FileSystem`、`Registry`、`Certificate`、`Environment`、`Variable`、`Alias`、`Function`）；PSDrive 将提供程序挂载到命名路径；提供程序特有的动态参数。
**语法/类/函数**：`Get-PSProvider`、`Get-PSDrive`、`New-PSDrive`、`Get-ChildItem Env:`、`Get-ItemProperty HKCU:\Software\...`。

### 10.2 文件系统、注册表与环境变量操作
**简介**：本节把最常用的三类提供程序操作集中呈现，但重点放在“什么时候应该用 cmdlet、什么时候应该直接调用 .NET 基类库”。
**知识点**：`New-Item` / `Copy-Item` / `Move-Item` / `Remove-Item` / `Test-Path` 的路径解析规则；`Get-Content` / `Set-Content` / `Add-Content` 的编码与流式读取；注册表键与值的区分（`Get-Item` vs `Get-ItemProperty`）；环境变量提供程序的进程级作用域。
**语法/类/函数**：`[System.IO.File]::WriteAllText()` 与 `Set-Content` 的选择边界；`Get-Content -ReadCount 0` 的整文件读取；`[System.Environment]::SetEnvironmentVariable()`。

## 第11章 管道深入：对象流、参数绑定与 `$input`

### 11.1 管道对象流与 `Begin` / `Process` / `End` 块
**简介**：管道中的每个 cmdlet/函数不是“接收整个集合”，而是以 `Begin` → 逐对象 `Process` → `End` 的生命周期运作；理解这一点是写出高效流式命令的前提。
**知识点**：`begin` / `process` / `end` 命名块在高级函数中的执行时机；`$input` 枚举器在非高级函数中的语义；`process` 块中 `$_` 的当前对象含义；`clean` 块（PowerShell 7.3+）在错误路径上的补充清理。
**语法/类/函数**：`function Measure-Lines { begin { $count = 0 } process { $count++ } end { $count } }`；`$input | ForEach-Object { ... }`。

### 11.2 参数绑定：按值、按属性名与延迟绑定
**简介**：管道输入绑定到参数有两条路径——“按值”（整个对象赋给参数）和“按属性名”（对象的某个属性赋给参数），延迟绑定脚本块只在后者且参数类型不是 `[object]` / `[scriptblock]` 时生效。
**知识点**：`ValueFromPipeline` 与 `ValueFromPipelineByPropertyName` 的声明方式；绑定失败时的回退行为；延迟绑定脚本块对每个传入对象逐次求值；`$PSBoundParameters` 查看实际绑定。
**语法/类/函数**：`[Parameter(ValueFromPipelineByPropertyName)] [string]$Name`；`Get-Process | Select-Object @{Name='ProcName'; Expression={ $_.Name }}` 中的计算属性绑定。

## 第12章 对象筛选、投影、排序与分组

### 12.1 `Where-Object` 的两种形态与性能取舍
**简介**：`Where-Object` 的简化语法（`-Property` / `-Value`）在简单比较时更短，但脚本块语法才能表达复合条件和属性方法调用。
**知识点**：`-EQ` / `-NE` / `-Like` / `-Match` 作为简化比较运算符；脚本块中 `$_` 与 `$PSItem` 的等价性；简化语法不支持逻辑组合的边界。
**语法/类/函数**：`Where-Object Status -EQ 'Running'` 与 `Where-Object { $_.Status -eq 'Running' -and $_.CPU -gt 10 }` 的对照。

### 12.2 `Select-Object`：属性投影与对象塑形
**简介**：`Select-Object` 不只是“选列”，它可以通过计算属性、`-First` / `-Last` / `-Skip` 和 `-ExpandProperty` 改变管道中对象的“形状”和数量。
**知识点**：`-Property` 接受属性名、哈希表（计算属性）和嵌套哈希表；`-ExpandProperty` 将单个属性值“展开”为管道输出；`-First` / `-Last` 的提前终止语义与对上游 cmdlet 的影响。
**语法/类/函数**：`Select-Object Name, @{Name='MemMB'; Expression={ [math]::Round($_.WorkingSet / 1MB, 2) }}`；`Select-Object -ExpandProperty Name`。

### 12.3 排序、分组与度量
**简介**：`Sort-Object`、`Group-Object`、`Measure-Object` 构成 PowerShell 的“聚合三件套”，对应 LINQ 的 `OrderBy`、`GroupBy` 和聚合运算符。
**知识点**：`Sort-Object` 的多属性排序（哈希表形式的 `@{Expression=...; Descending=$true}`）；`Group-Object` 的 `-NoElement` 与 `Count` 属性；`Measure-Object` 对非数值属性的计数与对文本的 `-Word` / `-Line`。
**语法/类/函数**：`Sort-Object @{Expression='Length'; Descending=$true}, Name`；`Group-Object -Property Status | Select-Object Name, Count`；`Measure-Object -Property Length -Sum -Average`。

### 12.4 格式化系统与输出流
**简介**：格式化 cmdlet 将对象转换为“格式化指令对象”，再由主机渲染；它们位于管道末端，不应与数据操作 cmdlet 混用。PowerShell 有多个独立输出流（Success、Error、Warning、Verbose、Debug、Information），重定向与文件输出需要明确“序列化”还是“渲染”。
**知识点**：`Format-Table` 的 `-AutoSize`、`-GroupBy`、`-Wrap`；`Format-List` 的 `-Property *` 全属性视图；格式化对象的“不可逆”性质——`Format-*` 之后的管道对象不再是原始数据对象；`>` 与 `Out-File` 的默认编码差异；`Export-Csv` / `Import-Csv` 的类型退化；`ConvertTo-Json` / `ConvertFrom-Json` 的深度与 `-AsHashtable`。
**语法/类/函数**：`Format-Table Name, CPU -AutoSize`；`Get-Process | Export-Csv -NoTypeInformation`；`ConvertTo-Json -Depth 10`；`[System.Management.Automation.FormatInfo]`。


# 第3部分 .NET 运行时设施：在 PowerShell 中使用 CLR

> **部分定位**：本部分是全书中与《CLR via C#》对照最紧密的部分。PowerShell 脚本不是“另一种语言运行时”，它直接运行在 CLR 上，因此程序集加载、类型反射、IO、网络、文本处理、容器、线程与图形界面这些 CLR 设施都可以从脚本中触及——关键在于知道“哪些 .NET 能力值得从 PowerShell 调用”以及“调用时的边界在哪里”。本部分按“基础设施（第13章）→ 数据与通信（第14–16章）→ 并发（第17章）→ 流与持久化（第18章）→ 图形界面（第19章）”组织。

## 第13章 程序集、类型发现与反射

### 13.1 程序集加载与 `Add-Type`
**简介**：对标《CLR via C#》第23章“程序集加载和反射”和第2–3章“生成/打包/部署程序集”，`Add-Type` 是 PowerShell 中“把新类型带入当前会话”的主要入口。
**知识点**：`Add-Type` 的四种输入源（`-TypeDefinition` C# 源码、`-Path` 源文件、`-AssemblyName` 已加载程序集、`-MemberDefinition` 成员定义）；类型一旦在会话中定义便不可卸载或重定义，同名类型需新会话或改名；`-OutputAssembly` 将动态生成的程序集持久化为 DLL。
**语法/类/函数**：`Add-Type -TypeDefinition 'public class X { ... }'`；`Add-Type -AssemblyName System.Windows.Forms`；`[System.Reflection.Assembly]::LoadFrom()`、`[System.Reflection.Assembly]::LoadWithPartialName()`（遗留）。

### 13.2 反射：`GetType()`、`GetMember()`、`InvokeMember()`
**简介**：PowerShell 的 `Get-Member` 是反射的“用户友好面”，但底层 `System.Reflection` 提供了更完整的类型元数据访问和后期绑定调用。
**知识点**：`$obj.GetType()` 返回运行时类型；`GetMember()` / `GetMethods()` / `GetProperties()` 的 `BindingFlags` 过滤；`InvokeMember()` 的后期绑定调用；`[type]::GetType()` 与程序集限定名。
**语法/类/函数**：`[System.Reflection.BindingFlags]::Public -bor [System.Reflection.BindingFlags]::Instance`；`$type.GetMethod('M').Invoke($obj, @(1, 2))`。

### 13.3 `New-Object`、`[type]::new()` 与构造函数选择
**简介**：创建 .NET 对象有两条等价路径，但它们在参数发现、性能和 COM 互操作上有明确的分工。
**知识点**：`New-Object` 通过 `-TypeName` 和 `-ArgumentList` 调用构造函数；`[type]::new()` 在 PowerShell 5.0+ 中可用，不传参时显示构造函数列表，适合探索式编码；创建 COM 对象只能使用 `New-Object -ComObject`。
**语法/类/函数**：`New-Object System.IO.FileInfo('x.txt')`；`[System.IO.FileInfo]::new('x.txt')`；`New-Object -ComObject WScript.Shell`。

## 第14章 网络：从应用层 HTTP 到传输层 Socket

> **章内定位**：.NET 在 `System.Net.*` 和 `System.Net.Sockets.*` 中提供了分层网络能力：应用层有 `WebClient`、`HttpClient`、`HttpListener`，传输层有 `TcpClient` / `TcpListener` / `UdpClient` / `Socket`。PowerShell 同时提供了 cmdlet 封装（`Invoke-RestMethod`、`Invoke-WebRequest`），但 cmdlet 与裸 .NET 类在响应形态、可配置性、异步模型上有明确分工。核心判断线索是：“取数据”优先用 cmdlet，“控协议/建服务/走 TCP”才下沉到 .NET 类。

### 14.1 `Invoke-RestMethod` / `Invoke-WebRequest` 与 `HttpClient` 的分工
**简介**：`Invoke-RestMethod` 自动将 JSON/XML 响应体反序列化为 PowerShell 对象，`Invoke-WebRequest` 返回带状态码和头部的响应对象；而 `HttpClient` 是需要显式管理生命周期、但可精细控制请求管道和异步行为的 .NET 类。对标《CLR via C#》的异步问题意识：`HttpClient` 的 `GetAsync` 在 PowerShell 中如何被“同步等待”。
**知识点**：`Invoke-RestMethod` 直接返回响应体（已对象化），`Invoke-WebRequest` 返回 `HtmlWebResponseObject` 且 `.Content` 恒为字符串；`HttpClient` 的 `GetAsync().Result` 在 PowerShell 中的阻塞等待与死锁风险；`HttpClientHandler` 对代理、证书验证的控制；`-SkipCertificateCheck` 在 PowerShell 7+ 中的可用性。
**语法/类/函数**：`Invoke-RestMethod -Uri $uri -Method Get`；`[System.Net.Http.HttpClient]::new()`、`.GetStringAsync()`、`.PostAsync()`；`[System.Net.Http.HttpClientHandler]::new()` 设置 `ServerCertificateCustomValidationCallback`。

### 14.2 `HttpListener`：在脚本中启动 HTTP 服务
**简介**：`HttpListener` 使 PowerShell 脚本可以充当轻量 HTTP 服务器，常用于回调接收、本地工具的服务端模拟和简易 API mock。对标《CLR via C#》的程序集宿主话题：脚本进程本身成为一个网络服务的宿主。
**知识点**：`HttpListener` 的 `Prefixes` 需以 `/` 结尾且需 URL ACL 或管理员权限绑定非回环地址；`GetContext()` 的阻塞语义与在 PowerShell 中如何配合 `Start-ThreadJob` 避免主线程冻结；请求体读取与响应写入的流路径。
**语法/类/函数**：`[System.Net.HttpListener]::new()`、`$listener.Prefixes.Add('http://localhost:8080/')`、`$listener.Start()`、`$ctx = $listener.GetContext()`；`[System.IO.StreamReader]($ctx.Request.InputStream)` 读取请求体。

### 14.3 `TcpClient` / `TcpListener` / `UdpClient`：传输层直接控制
**简介**：当需要实现非 HTTP 协议、端口探测或原始套接字通信时，必须越过 cmdlet 层直接使用 `System.Net.Sockets`。对标《CLR via C#》的异步 I/O 章节：`NetworkStream` 的读写本质上是流操作，与第18章的 IO 设施是同一套抽象。
**知识点**：`TcpListener` 的 `Start()` / `AcceptTcpClient()` 阻塞等待连接；`TcpClient` 的 `GetStream()` 返回 `NetworkStream`，可与 `StreamReader` / `StreamWriter` 组合进行文本协议通信；`UdpClient` 的无连接语义与 `Receive()` 的阻塞性；`Socket` 在需要设置 `SocketOption`（超时、缓冲区大小、`NoDelay`）时的必要性。
**语法/类/函数**：`[System.Net.Sockets.TcpListener]::new([System.Net.IPAddress]::Any, 9000)`、`$listener.AcceptTcpClient()`；`[System.Net.Sockets.TcpClient]::new($host, $port)`、`$client.GetStream()`；`[System.Net.Sockets.UdpClient]::new($port)`、`.Receive()`。

### 14.4 地址、端点与 `Uri` 的底层表示
**简介**：网络编程中地址和端口不是字符串，而是 `IPAddress`、`IPEndPoint` 和 `Uri` 对象。PowerShell 的字符串宽松性在地址解析上会隐藏错误，直接使用这些类型可以获得早期失败。
**知识点**：`IPAddress.Parse()` 与构造函数接受字节数组的差异；IPv6 地址在字符串表示中需方括号包裹；`IPEndPoint` 的 `Address` + `Port` 组合；`Uri` 的 `Scheme`、`Host`、`Port`、`LocalPath`、`AbsolutePath`、`IsLoopback` 等属性对 URI 的结构化解析；`Dns.GetHostAddresses()` 的域名解析路径。
**语法/类/函数**：`[System.Net.IPAddress]::Parse()`、`[System.Net.IPEndPoint]::new($ip, $port)`；`[System.Uri]::new()`、`$uri.IsLoopback`、`$uri.LocalPath`；`[System.Net.Dns]::GetHostAddresses($host)`。

## 第15章 文本与正则：`String` 不可变性、`StringBuilder` 与 `RegularExpressions`

> **章内定位**：对标《CLR via C#》第14章“字符、字符串和文本处理”。PowerShell 表面上让字符串操作“很轻松”，但 `-match` / `-replace` 的隐式行为（大小写不敏感、`$matches` 副作用、替换字符串中 `$` 的变量插值冲突）恰恰是脚本 bug 的高发区。本章的教学目标是：用 .NET 的显式类型纠正 PowerShell 运算符的“方便但不精确”。

### 15.1 字符串不可变性与 `StringBuilder`
**简介**：`System.String` 在 CLR 中不可变，每次拼接都分配新对象；PowerShell 的 `+=` 在循环中尤其昂贵。对标《CLR via C#》对字符串驻留和分配的讨论，`StringBuilder` 是唯一能在高频拼接场景中保持线性复杂度的 .NET 设施。
**知识点**：字符串字面量的驻留（interning）与 `[string]::IsInterned()`；`+=` 在循环中的 O(n²) 行为与 `StringBuilder.Append()` 的摊销 O(1)；`StringBuilder` 的 `Capacity` 预分配对大规模构建的收益；`[System.String]::Concat()` / `Join()` 在已知有限片段时的适用性。
**语法/类/函数**：`[System.Text.StringBuilder]::new($capacity)`、`.Append()`、`.AppendLine()`、`.ToString()`；`[string]::Join($sep, $array)`。

### 15.2 `-match` / `-replace` / `-split` 与 `Regex` 类的语义对照
**简介**：PowerShell 的 `-match` 默认大小写不敏感（与 .NET `Regex` 默认敏感相反），并会设置 `$matches` 自动变量；`-replace` 的替换字符串在双引号中会先经过 PowerShell 变量插值，然后才交给 `Regex.Replace()`——这是 `$1` 在 PowerShell 替换模式中“不生效”的根本原因。
**知识点**：`-match` 的 `-imatch` / `-cmatch` 前缀显式控制敏感性；`$matches[0]` 为整体匹配，`$matches[1]` 为第一捕获组，`$matches['name']` 为命名组；`-replace` 的替换占位符 `$&`（整体匹配）、`$1`、`${name}` 在单引号替换字符串中才可安全使用；`Select-String` 返回的 `MatchInfo` 与 `[regex]::Matches()` 返回的 `MatchCollection` 之间的类型差异——前者是 PowerShell 包装，后者是裸 .NET 集合。
**语法/类/函数**：`'text' -match 'pattern'`、`$matches`；`[regex]::Matches($input, $pattern)` 返回 `MatchCollection`；`[regex]::Replace($input, $pattern, '${name}')`；`[System.Text.RegularExpressions.RegexOptions]::IgnoreCase`。

### 15.3 动态正则、超时与灾难性回溯
**简介**：当模式来自变量时，`[regex]::Escape()` 是防止元字符注入的第一道防线；而复杂模式在不可信输入上可能触发灾难性回溯，`Regex` 构造函数提供了 `TimeSpan` 超时参数来为脚本设置硬性上限。
**知识点**：`[regex]::Escape()` 对字面量模式的必要性；`Regex` 的 `matchTimeout` 构造参数（PowerShell 默认的 `-match` 没有超时保护）；命名捕获组 `(?<name>...)` 在提取结构化字段时的可读性优势；`RegexOptions.Compiled` 在 PowerShell 脚本中收益有限（单次执行场景下编译成本可能超过收益）。
**语法/类/函数**：`[regex]::Escape($literal)`；`[System.Text.RegularExpressions.Regex]::new($pattern, $options, [timespan]::FromSeconds(1))`；`$regex.Match($input)`、`$regex.IsMatch($input)`。

### 15.4 区域性敏感的文本处理：`CultureInfo` 与 `IFormattable`
**简介**：对标《CLR via C#》中“格式化器”与“区域性”的讨论。PowerShell 在多数场景下以 invariant culture 进行字符串化，但显式调用 `ToString(format, provider)` 时可以使用任意 `CultureInfo`；这一“隐式 invariant、显式可控”的二元性直接影响日期和数字的脚本输出是否可跨区域复现。
**知识点**：PowerShell 的隐式字符串化通常使用 `[cultureinfo]::InvariantCulture`；`IFormattable.ToString(format, provider)` 是让类型接受格式字符串和区域性的契约接口；`DateTime`、数值类型和 `enum` 都实现了 `IFormattable`；`CultureInfo.CurrentCulture` 与 `InvariantCulture` 在 `ToString()` 调用中的选择对脚本可移植性的影响。
**语法/类/函数**：`$dt.ToString('yyyy-MM-dd', [cultureinfo]::InvariantCulture)`；`[System.IFormattable]` 接口；`[cultureinfo]::GetCultureInfo('de-DE')`；`(1234.5).ToString([cultureinfo]::InvariantCulture)`。

## 第16章 容器：从 `ArrayList` 的遗留教训到并发集合

> **章内定位**：对标《C# 本质论》“泛型集合”章和《CLR via C#》第12章“泛型”。PowerShell 的数组是固定大小的，`+=` 每次创建新数组，在循环中性能急剧退化。本章按“顺序容器 → 关联容器 → 并发容器”三层组织，核心判断线是：单线程用泛型集合，多线程/并行脚本块必须用 `System.Collections.Concurrent`。

### 16.1 顺序容器：`ArrayList` vs `List<T>` 的选型与代价
**简介**：`ArrayList` 是非泛型的遗留类型，混入不同类型时不会阻止，但读取时需要强制转换且可能产生意外返回类型；`List<T>` 是类型安全的泛型替代，在 PowerShell 中通过 `[System.Collections.Generic.List[T]]` 实例化。对标《CLR via C#》的装箱/拆箱讨论：非泛型集合对值类型会产生装箱开销。
**知识点**：`ArrayList.Add()` 返回索引值，在管道中若不抑制（`[void]` 或 `$null =`）会污染输出流；`List<T>` 的 `Add()` 返回 `void`，行为更干净；`AddRange()` 批量添加；`FindAll()` 接受 `Predicate<T>` 委托，在 PowerShell 中可用 `ScriptBlock` 传入；`Sort()` 与 `Reverse()` 的就地修改语义。
**语法/类/函数**：`[System.Collections.Generic.List[string]]::new()`、`.Add()`、`.AddRange()`、`.FindAll({ param($x) ... })`；`[System.Collections.ArrayList]::new()` 的 `[void]` 抑制。

### 16.2 关联容器：`Hashtable`、`Dictionary<TKey,TValue>` 与 `[ordered]`
**简介**：PowerShell 字面量 `@{}` 产生 `Hashtable`，`[ordered]@{}` 产生 `OrderedDictionary`；而泛型 `Dictionary<TKey,TValue>` 在大量键值查询场景中提供了类型安全和性能优势。对标《CLR via C#》对“哈希表”和“相等性”的讨论：键的 `GetHashCode` / `Equals` 行为决定了查找正确性。
**知识点**：`Hashtable` 的键为 `object`，允许混用类型但可能导致哈希冲突异常；`Dictionary<string,object>` 的强类型键；`[ordered]` 保持插入顺序但在大规模插入时性能低于 `Dictionary`；`TryGetValue()` 避免“键不存在则抛异常”的异常驱动控制流；`ContainsKey()` 在 PowerShell 中常被用于替代 `$hash[$key]` 的静默 `$null`。
**语法/类/函数**：`[System.Collections.Generic.Dictionary[string,int]]::new()`、`.TryGetValue($key, [ref]$value)`；`[ordered]@{}` 的 `.Keys` 保持插入顺序。

### 16.3 并发集合：`ConcurrentDictionary` 与 `BlockingCollection`
**简介**：PowerShell 的管道和 `ForEach-Object -Parallel` 中的脚本块在不同线程上执行，普通 `List<T>` 和 `Hashtable` 的并发写入会产生竞态和损坏。`System.Collections.Concurrent` 命名空间提供了无锁/细粒度锁的线程安全集合，是在并行脚本中收集结果和实现生产者-消费者模式的唯一正确基础设施。
**知识点**：`ConcurrentDictionary<TKey,TValue>` 的 `AddOrUpdate()` / `GetOrAdd()` 的原子语义，以及跨脚本块共享同一实例的方式；`BlockingCollection<T>` 的 `Add()` / `Take()` 阻塞语义与有界容量（构造参数 `boundedCapacity`），用于并行生产者与消费者的解耦；`ConcurrentQueue<T>` / `ConcurrentStack<T>` 的 `Enqueue` / `TryDequeue` 无阻塞模式；**PowerShell 边界**：`$using:` 只传递只读快照，并发集合必须作为引用类型通过闭包或参数传入脚本块，而不是依赖 `$using:` 复制。
**语法/类/函数**：`[System.Collections.Concurrent.ConcurrentDictionary[string,object]]::new()`、`.AddOrUpdate()`；`[System.Collections.Concurrent.BlockingCollection[int]]::new(5)`、`.Add()`、`.Take()`、`.CompleteAdding()`。

## 第17章 线程与并发：`Thread` / `ThreadPool` / `Task` 与 PowerShell 运行空间

> **章内定位**：对标《CLR via C#》第26–30章“线程基础/计算限制的异步操作/I/O 限制的异步操作”。PowerShell 的并发栈有四层：进程隔离的 `Start-Job` → 线程级 `Start-ThreadJob` → 管道并行 `ForEach-Object -Parallel` → 裸 `Runspace` API 和最底层的 .NET `Thread` / `ThreadPool` / `Task`。本章的写作原则是：先讲清每层的抽象代价和隔离边界，再落到 .NET 原语。

### 17.1 `Thread`、`ThreadPool` 与 `Task` 在 PowerShell 中的可用性与边界
**简介**：PowerShell 可以直接构造 `[System.Threading.Thread]` 和调用 `[System.Threading.Tasks.Task]::Run()`，但这通常不是正确选择：PowerShell 的执行依赖运行空间的线程关联性，一个运行空间同一时刻只能在一个线程上活动；在裸 `Thread` 中直接执行 PowerShell 代码而没有关联运行空间会导致状态丢失或 `PSInvalidOperationException`。
**知识点**：`Thread` 是 OS 级原语，创建成本高且需手动 `Start()` / `Join()`；`ThreadPool` 回收工作线程，`Task.Run` 是其上的标准抽象；在 PowerShell 中，`Task` 适合包装单个 .NET 方法调用（如异步 I/O），而不是执行 `ScriptBlock`；`[System.Threading.Thread]::CurrentThread.ApartmentState` 与后续 WinForm 的 STA 要求直接相关。
**语法/类/函数**：`[System.Threading.Thread]::new([System.Threading.ThreadStart]{ ... })`；`[System.Threading.Tasks.Task]::Run([Action]{ ... })`、`.Wait()`、`.Result`；`[System.Threading.ThreadPool]::QueueUserWorkItem()`。

### 17.2 `Start-ThreadJob`、`ForEach-Object -Parallel` 与 `Runspace` 的选型
**简介**：`Start-Job` 启动独立进程（隔离好、开销大），`Start-ThreadJob` 在 .NET 线程上运行（启动快、资源小），`ForEach-Object -Parallel` 提供管道内并行，`Runspace` API 提供最细粒度的控制和池化。对标《CLR via C#》的线程池直觉：`-ThrottleLimit` 对应的是工作线程数量的上限。
**知识点**：`ForEach-Object -Parallel` 的 `-ThrottleLimit` 控制并发度；`$using:` 变量在并行脚本块中是只读快照，修改不会回传；`Start-ThreadJob` 的 `Receive-Job` 收集输出与 `Remove-Job` 清理责任；`[runspacefactory]::CreateRunspacePool()` 的 `Open()` / `CreatePowerShell()` / `BeginInvoke()` 异步模式；**PowerShell 边界**：并行脚本块中不要直接操作共享的 `List<T>` 或 `Hashtable`，必须使用第16章的并发集合。
**语法/类/函数**：`1..10 | ForEach-Object -Parallel { $_ * 2 } -ThrottleLimit 5`；`Start-ThreadJob -ScriptBlock { ... }`；`[System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspacePool(1, 5)`。

### 17.3 同步原语：`Monitor`、`Interlocked` 与异步等待
**简介**：PowerShell 没有 `lock` 关键字，当并行脚本块必须访问非并发集合或共享状态时，需要显式调用 .NET 的同步原语。对标《CLR via C#》的“基元线程同步构造”：`Interlocked` 适合单一数值的原子操作，`Monitor` 适合临界区，`SemaphoreSlim` 适合限制并发访问数。
**知识点**：`[System.Threading.Monitor]::Enter($obj)` / `Exit($obj)` 的 `try/finally` 配对要求；`[System.Threading.Interlocked]::Increment()` / `Add()` 的无锁原子性；`[System.Threading.SemaphoreSlim]::Wait()` / `Release()` 的异步友好性；`[System.Threading.Mutex]` 的跨进程命名互斥体（适用于多脚本实例协调）。
**语法/类/函数**：`[System.Threading.Interlocked]::Increment([ref]$counter)`；`[System.Threading.Monitor]::Enter($lockObj)`；`[System.Threading.SemaphoreSlim]::new($initial, $max)`。

## 第18章 IO 深化：流、编码与大文件处理

> **章内定位**：第10章已在“提供程序”层面讨论了 cmdlet 级的文件操作。本章处理 cmdlet 无法覆盖的场景：大文件流式读取、显式编码控制、`Stream` 的组合与包装、管道与流的桥接。对标《CLR via C#》第14章中“流”作为字节抽象与字符抽象的层次关系：`FileStream` 是字节流，`StreamReader` / `StreamWriter` 是字符层的适配器。

### 18.1 `StreamReader` / `StreamWriter` 与 `File.ReadLines()` 的流式语义
**简介**：`Get-Content` 适合交互，但在大文件上一次性加载会耗尽内存。`[System.IO.File]::ReadLines()` 返回延迟求值的 `IEnumerable<string>`，配合 `foreach` 可以逐行流式处理而不将整个文件载入内存；`StreamReader` 则在需要控制缓冲区大小和编码时使用。
**知识点**：`File.ReadLines()` 的延迟执行与 `foreach` 的配合（每一步只取一行）；`StreamReader` 的 `ReadLine()` 循环与 `EndOfStream` 判断；`StreamReader` 必须在 `finally` 中 `Dispose()` 或使用 `using`（在 PowerShell 中对应 `try/finally`）；`ReadToEnd()` 仅适用于确定规模的小文件。
**语法/类/函数**：`foreach ($line in [System.IO.File]::ReadLines($path)) { ... }`；`[System.IO.StreamReader]::new($path)`、`.ReadLine()`、`.EndOfStream`；`[System.IO.StreamWriter]::new($path, $false, [System.Text.Encoding]::UTF8)`。

### 18.2 编码、`Stream` 组合与 `FileStream`
**简介**：文本文件的读写错误中相当比例源于编码不匹配。`FileStream` 提供字节级读写，`StreamReader` / `StreamWriter` 在其上叠加字符编码层；PowerShell 的 `Get-Content` / `Set-Content` 的 `-Encoding` 参数是便利封装，但在精细场景下需要直接指定 `Encoding` 对象。
**知识点**：`FileStream` 的 `FileMode` / `FileAccess` / `FileShare` 组合对并发写入和文件锁的影响；`StreamReader` 构造函数接受 `Encoding`、`detectEncodingFromByteOrderMarks`、`bufferSize`；UTF-8 无 BOM 与带 BOM 在跨平台脚本中的兼容性问题；`MemoryStream` 作为内存中的流适配器在将字节数组转为流式接口时的作用。
**语法/类/函数**：`[System.IO.FileStream]::new($path, 'Open', 'Read', 'ReadWrite')`；`[System.Text.Encoding]::UTF8`、`[System.Text.Encoding]::GetEncoding('utf-8')`；`[System.IO.MemoryStream]::new($bytes)`。

### 18.3 管道与流的桥接：`$input`、`Stream` 与外部命令
**简介**：PowerShell 管道是对象流，`System.IO.Stream` 是字节流，两者之间没有自动桥接。当需要在脚本中将 cmdlet 的输出送入期望 `Stream` 的 .NET API，或反过来将 `Stream` 的内容逐项送入管道时，需要显式的适配层。
**知识点**：`$input` 在 `process` 块中的逐对象枚举与 `Stream` 的逐字节/逐字符语义的不匹配；将外部命令输出通过 `[System.IO.StreamReader]` 的 `BaseStream` 读取；`[System.IO.Pipes.PipeStream]` 在进程间通信脚本中的可用性；**PowerShell 边界**：不要把管道对象直接 `Cast` 到 `Stream`，也不要在管道中期望 `Stream` 的延迟读取语义——管道的同步推送模型与流的拉取模型是对立的。
**语法/类/函数**：`$proc = [System.Diagnostics.Process]::Start($psi); $proc.StandardOutput.BaseStream`；`[System.IO.Pipes.NamedPipeServerStream]::new('name')`。

## 第19章 图形界面：Windows Forms 与 PowerShell 的 STA 约束

> **章内定位**：Windows Forms 是 .NET Framework 时代的 UI 框架，但它在 PowerShell 脚本工具化中仍有不可替代的位置：无需创建完整项目即可为脚本快速添加输入框、进度窗口和文件选择对话框。本章的核心约束不是控件 API 本身，而是 PowerShell 默认的线程单元（MTA）与 WinForms 要求的 STA 之间的冲突——不理解 STA 和消息泵，WinForms 在 PowerShell 中就会表现为“窗口一闪而过”或“脚本假死”。

### 19.1 程序集加载、`Form` 与基本控件构建
**简介**：一个 PowerShell WinForms 脚本的最小骨架是：`Add-Type` 加载 `System.Windows.Forms` 和 `System.Drawing` → 创建 `Form` → 创建控件 → 加入 `$form.Controls` → 显示。控件属性（`Location`、`Size`、`Text`）通过 `System.Drawing.Point` / `Size` 设置，与 C# 的设计器生成代码在本质上是同一套 API。
**知识点**：`Add-Type -AssemblyName System.Windows.Forms` 和 `System.Drawing` 的必需性；`Form` 的 `Text`（标题）、`Size`、`StartPosition`（`CenterScreen`）；`Button`、`Label`、`TextBox` 的 `Location` / `Size` / `Text` 属性；`$form.Controls.Add()` 的扁平添加方式（无布局管理器时的绝对定位）；`AcceptButton` / `CancelButton` 对回车和 Esc 的绑定。
**语法/类/函数**：`New-Object System.Windows.Forms.Form`；`[System.Drawing.Point]::new(75,120)`、`[System.Drawing.Size]::new(75,23)`；`$form.Controls.Add($button)`。

### 19.2 `ShowDialog()`、STA 模式与消息泵
**简介**：`ShowDialog()` 以模态方式显示窗体，它会阻塞 PowerShell 脚本的执行直到窗体关闭，但它同时从操作系统获得消息泵，因此控件事件能够被处理；`Show()` 是非模态的，但 PowerShell 控制台线程默认不提供消息泵，导致窗体无响应或立即消失。此外，PowerShell 7 默认以 MTA 运行，而 WinForms 要求 STA，必须用 `pwsh -STA` 启动才能可靠工作。
**知识点**：`ShowDialog()` 的模态阻塞是脚本工具场景下的正确选择；`[System.Windows.Forms.Application]::Run($form)` 是替代消息循环，但会接管脚本控制流；`[System.Windows.Forms.Application]::DoEvents()` 在需要“不阻塞 UI 的同时继续脚本”时的谨慎使用；`pwsh -STA` 与 `powershell.exe`（Windows PowerShell 默认 STA）的差异；检测当前 `ApartmentState` 的方式。
**语法/类/函数**：`$result = $form.ShowDialog()`；`[System.Windows.Forms.Application]::Run($form)`；`[System.Threading.Thread]::CurrentThread.ApartmentState`；`pwsh -STA -File .\gui.ps1`。

### 19.3 事件绑定：`Add_<EventName>` 与 `ScriptBlock` 作为事件处理器
**简介**：PowerShell 通过 `Add_<EventName>({ ScriptBlock })` 模式将脚本块绑定到 .NET 控件事件，例如 `$button.Add_Click({ ... })`、`$form.Add_Shown({ $textBox.Select() })`。这是 `ScriptBlock` 在 WinForms 场景中作为 `EventHandler` 委托的实用形态，但事件处理器运行在 UI 线程上，不能在其中执行长时间阻塞操作。
**知识点**：`Add_Click` / `Add_Shown` / `Add_FormClosing` 的命名约定；事件处理器中的 `$this` 指代触发事件的对象（PowerShell 的自动变量，与 C# 的 `sender` 对应）；在 `Add_Click` 中访问外部变量（`$textBox`、`$form`）的脚本作用域捕获；事件处理器中不要调用 `ShowDialog()` 阻塞 UI 线程。
**语法/类/函数**：`$button.Add_Click({ $result = $textBox.Text; $form.Close() })`；`$form.Add_Shown({ $textBox.Select() })`；`$this.Text`。

### 19.4 常用控件组合与“输入框 / 文件选择”模式
**简介**：脚本工具中最常见的 WinForms 用途不是构建完整应用，而是两个模式：自定义输入框（`Label` + `TextBox` + `OK` / `Cancel` + `ShowDialog()` 返回值判断）和文件/文件夹选择（`OpenFileDialog` / `FolderBrowserDialog`）。这两个模式可以在 20–30 行 PowerShell 内完成，产出可复用的函数。
**知识点**：`ShowDialog()` 返回 `[System.Windows.Forms.DialogResult]::OK` / `Cancel` 的判断；`OpenFileDialog` 的 `Filter`、`Multiselect`、`InitialDirectory`；`FolderBrowserDialog` 的 `SelectedPath`；`SaveFileDialog` 的 `DefaultExt` / `AddExtension`；将整个模式封装为 `function Show-InputBox` / `function Select-File` 以便在脚本中复用。
**语法/类/函数**：`[System.Windows.Forms.OpenFileDialog]::new()`、`.ShowDialog()`、`.FileName`；`[System.Windows.Forms.FolderBrowserDialog]::new()`、`.SelectedPath`；`[System.Windows.Forms.DialogResult]::OK`。


# 第4部分 Win32 API 编程：系统机制与用户界面

> **部分定位**：本部分以《Windows 核心编程》的“系统机制”框架和《Windows程序设计》的“窗口/消息/图形”框架为组织蓝本，但始终站在 PowerShell 脚本的立场上。全书的总判断线是：**Win32 API 在 PowerShell 中的价值，不在于“重写 C++ 程序”，而在于触及 .NET 基类库和 cmdlet 没有封装、或封装得不够底层的机制**——内核对象句柄的等待与信号、线程同步的精确控制、虚拟内存的查询与提交、DLL 的显式加载、窗口消息的发送与拦截、以及 GDI 设备上下文的绘制。每一章都在回答同一个问题：**.NET 已经提供了什么，Win32 还剩下什么不可替代，以及调用它的代价是什么**。

## 第20章 基础必备知识：错误处理、字符字符串与内核对象

> **章内定位**：对应《Windows 核心编程》第I部分“必备知识”（错误处理、字符和字符串处理、内核对象）。这三项是后续所有 Win32 调用的共同基础设施：不声明 `SetLastError` 就无法诊断失败，不指定 `CharSet` 就会在 ANSI/Unicode 之间静默出错，不理解内核对象句柄的“可等待/可关闭”语义就无法安全地跨 API 管理资源。PowerShell 的 `Add-Type` 和 `Marshal` 是这套基础设施在脚本层的实现载体。

### 20.1 错误处理：`GetLastError`、`SetLastError` 与 `Win32Exception`
**简介**：对标《Windows 核心编程》第1章“错误处理”。Win32 API 的错误报告机制是“返回失败 + 设置线程的最后错误码”。在 P/Invoke 签名中声明 `SetLastError = true` 后，失败时应**立即**通过 `[Marshal]::GetLastWin32Error()` 读取错误码，再转换为可读消息。延迟读取会被后续 P/Invoke 调用覆盖。
**知识点**：`SetLastError = true` 是错误码可靠获取的前提；`[Marshal]::GetLastWin32Error()` 必须紧随失败调用；`[System.ComponentModel.Win32Exception]::new($code).Message` 将错误码转为系统消息；PowerShell 的 `$?` 对非布尔返回值的 P/Invoke 调用不可靠，不能替代显式错误检查。
**语法/类/函数**：`[DllImport("kernel32.dll", SetLastError = true)] public static extern bool SomeApi(...);`；`$ok = [Win32]::SomeApi(...); if (-not $ok) { $code = [Marshal]::GetLastWin32Error(); [System.ComponentModel.Win32Exception]::new($code).Message }`。

### 20.2 字符与字符串封送：`CharSet`、`StringBuilder` 与 `Marshal`
**简介**：对标《Windows 核心编程》第2章“字符和字符串处理”。Win32 API 的字符串参数有 ANSI（`A` 后缀）和 Unicode（`W` 后缀）两个版本；`CharSet.Auto` 在 Windows NT 系上映射到 `W`。PowerShell 中封送字符串的标准路径是：输入字符串直接传 `string`，输出缓冲区用 `StringBuilder`，非托管往返用 `Marshal.StringToHGlobalAuto()` / `PtrToStringAuto()`。
**知识点**：`CharSet = CharSet.Auto` 的跨版本行为；输出缓冲区必须用 `StringBuilder` 并预分配容量，不能传 `string`（字符串不可变）；`Marshal.StringToHGlobalAuto()` 分配的非托管字符串必须在 `finally` 中 `FreeHGlobal`；`[Marshal]::PtrToStringAuto()` 将非托管指针转回托管字符串。
**语法/类/函数**：`[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)] public static extern uint GetModuleFileName(IntPtr hModule, StringBuilder buffer, uint size);`；`$sb = [System.Text.StringBuilder]::new(260); [Win32]::GetModuleFileName([IntPtr]::Zero, $sb, 260)`。

### 20.3 内核对象与句柄：`IntPtr`、`CloseHandle` 与可等待语义
**简介**：对标《Windows 核心编程》第3章“内核对象”。进程、线程、文件、事件、互斥体、信号量在 Win32 中都是**内核对象**，通过句柄（`HANDLE`，在 PowerShell 中为 `IntPtr`）引用。每个内核对象都有一个使用计数；`CloseHandle` 递减计数，计数归零时对象销毁。句柄的“可等待”（signaled / non-signaled）状态是第23章同步机制的基础。
**知识点**：`[IntPtr]::Zero` 作为 `NULL` 句柄；内核对象句柄不是普通整数，**不可用 `[int]` 强转**（64 位下会截断）；`CloseHandle` 必须在 `finally` 中调用；`WaitForSingleObject` 接受任何可等待内核对象句柄；`GetLastError` 在 `CloseHandle` 失败时同样可用。
**语法/类/函数**：`[DllImport("kernel32.dll", SetLastError = true)] public static extern bool CloseHandle(IntPtr handle);`；`$h = [Win32]::CreateEvent(...); try { ... } finally { [Win32]::CloseHandle($h) }`；`[System.Threading.WaitHandle]::WaitAny()` 在 .NET 层等待多个句柄的替代路径。

### 20.4 结构体封送：`StructLayout` 与 `Marshal.PtrToStructure`
**简介**：对标《Windows 核心编程》中大量结构体 API 的调用模式。`SYSTEM_INFO`、`STARTUPINFO`、`PROCESSENTRY32` 等都需要在 `Add-Type -TypeDefinition` 中定义 `[StructLayout(LayoutKind.Sequential)]` 的 C# 结构体，用 `Marshal.SizeOf()` 分配非托管内存，调用 API 填充，再用 `Marshal.PtrToStructure()` 转换回托管对象。
**知识点**：`[StructLayout(LayoutKind.Sequential)]` 的顺序布局；`[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]` 对定长字符数组字段的声明；`cb` 字段通常必须先赋值为 `Marshal.SizeOf()`；`try/finally` 中 `FreeHGlobal` 的强制配对。
**语法/类/函数**：`[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)] public struct STARTUPINFO { public int cb; ... }`；`$size = [Marshal]::SizeOf([type][STARTUPINFO]); $ptr = [Marshal]::AllocHGlobal($size); [Marshal]::PtrToStructure($ptr, [type][STARTUPINFO])`。

## 第21章 进程与作业

> **章内定位**：对应《Windows 核心编程》第4–5章“进程”与“作业”。PowerShell 的 `Start-Process`、`Get-Process` 和 `[System.Diagnostics.Process]` 已经覆盖了进程的创建与查询，但 Win32 路径在**挂起启动、句柄继承控制、作业对象（Job Object）的 CPU/内存限制**这些场景下不可替代。

### 21.1 进程创建：`CreateProcess` 与 `STARTUPINFO`
**简介**：对标《Windows 核心编程》第4章“进程”。`CreateProcess` 返回 `PROCESS_INFORMATION`（进程句柄、线程句柄、PID、TID）。PowerShell 中 90% 的进程启动用 `Start-Process` 即可；`CreateProcess` 的必要性集中在：需要以 `CREATE_SUSPENDED` 状态创建进程、需要指定 `STARTUPINFO` 的桌面/窗口站、或需要获取进程句柄进行后续内存操作。
**知识点**：`STARTUPINFO.cb` 必须先赋值为结构体大小；`PROCESS_INFORMATION` 的 `hProcess` / `hThread` 使用后必须 `CloseHandle`；`CREATE_SUSPENDED` 与 `ResumeThread` 的配合；`CreateProcessAsUser` / `CreateProcessWithLogonW` 在需要不同安全上下文时的使用。
**语法/类/函数**：`[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)] public static extern bool CreateProcess(string app, string cmd, IntPtr pa, IntPtr ta, bool inherit, uint flags, IntPtr env, string dir, ref STARTUPINFO si, out PROCESS_INFORMATION pi);`；`[DllImport("kernel32.dll")] public static extern uint ResumeThread(IntPtr hThread);`。

### 21.2 作业对象：`CreateJobObject` 与资源限制
**简介**：对标《Windows 核心编程》第5章“作业”。作业对象将一组进程作为一个单元管理，可以设置**进程数上限、CPU 时间上限、内存上限**，并在作业关闭时终止所有关联进程。PowerShell 中没有作业对象的直接 cmdlet，Win32 路径是唯一选择。
**知识点**：`CreateJobObject` 创建作业内核对象；`SetInformationJobObject` 设置 `JOBOBJECT_EXTENDED_LIMIT_INFORMATION`（内存限制、进程数限制）；`AssignProcessToJobObject` 将进程加入作业；`TerminateJobObject` 终止作业内所有进程。
**语法/类/函数**：`[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)] public static extern IntPtr CreateJobObject(IntPtr attrs, string name);`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern bool SetInformationJobObject(IntPtr job, int infoClass, IntPtr info, uint len);`。

## 第22章 线程基础与调度

> **章内定位**：对应《Windows 核心编程》第6–7章“线程基础”与“线程调度、优先级和亲和性”。PowerShell 的并发模型（运行空间、`ForEach-Object -Parallel`）是**任务级**抽象，而 Win32 线程 API 提供的是**线程级**控制：创建挂起线程、设置优先级、绑定 CPU 亲和性。本章的价值在于揭示“PowerShell 的并行脚本块之下，操作系统线程发生了什么”。

### 22.1 线程创建与 `CreateThread` 的 PowerShell 边界
**简介**：`CreateThread` 创建操作系统线程并返回句柄。**在 PowerShell 中直接调用 `CreateThread` 执行 PowerShell 代码是不安全的**——PowerShell 的执行依赖运行空间的线程关联性，裸线程没有关联运行空间会导致状态丢失。Win32 线程创建在 PowerShell 中的合理用途是：创建一个执行**纯 Win32 回调或 .NET 方法**的线程，而非 `ScriptBlock`。
**知识点**：`CreateThread` 的 `lpStartAddress` 是函数指针，在 PowerShell 中需要 `delegate` 配合 `Marshal.GetFunctionPointerForDelegate()`；`CREATE_SUSPENDED` 标志；`ResumeThread` 恢复；线程函数返回后线程终止，`CloseHandle` 仍需调用；PowerShell 的 `Start-ThreadJob` 是更安全的线程级抽象（参见第17章）。
**语法/类/函数**：`[DllImport("kernel32.dll", SetLastError = true)] public static extern IntPtr CreateThread(IntPtr attrs, uint stackSize, IntPtr startAddr, IntPtr param, uint flags, out uint threadId);`；`[System.Runtime.InteropServices.Marshal]::GetFunctionPointerForDelegate($delegate)`。

### 22.2 线程优先级与亲和性：`SetThreadPriority` / `SetThreadAffinityMask`
**简介**：对标《Windows 核心编程》第7章“线程调度、优先级和亲和性”。`SetThreadPriority` 调整线程相对进程优先级的偏移；`SetThreadAffinityMask` 将线程限制在指定的 CPU 核心集合上运行。PowerShell 本身没有暴露这些控制，Win32 路径用于**性能敏感脚本的 CPU 绑核**或**实时性调试**。
**知识点**：`GetCurrentThread()` 返回伪句柄（`-2`），不需要 `CloseHandle`；`THREAD_PRIORITY_HIGHEST` / `THREAD_PRIORITY_BELOW_NORMAL` 等常量；`SetThreadAffinityMask` 的掩码位对应逻辑处理器；`GetSystemInfo` 的 `dwActiveProcessorMask` 提供可用 CPU 集合。
**语法/类/函数**：`[DllImport("kernel32.dll")] public static extern IntPtr GetCurrentThread();`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern bool SetThreadPriority(IntPtr hThread, int priority);`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern UIntPtr SetThreadAffinityMask(IntPtr hThread, UIntPtr mask);`。

## 第23章 线程同步：用户模式与内核对象

> **章内定位**：对应《Windows 核心编程》第8–9章“用户模式下的线程同步”与“用内核对象进行线程同步”。PowerShell 没有 `lock` 关键字，第17章已经介绍了 `Monitor` / `Interlocked` / `SemaphoreSlim`。本章补充的是**Win32 特有的同步原语**：`CRITICAL_SECTION`（用户模式临界区，比 `Monitor` 更轻）、`SRWLOCK`（读写锁）、以及可跨进程的命名 Mutex / Event / Semaphore。理解“用户模式同步”与“内核对象同步”的分界，是写出正确并行 PowerShell 代码的关键。

### 23.1 用户模式同步：`CRITICAL_SECTION` 与 `SRWLOCK`
**简介**：`CRITICAL_SECTION` 是用户模式的自旋+等待锁，初始化后在使用前 `EnterCriticalSection`，使用后 `LeaveCriticalSection`，不再需要时 `DeleteCriticalSection`。`SRWLOCK` 提供共享读/独占写的读写锁语义。两者都在用户模式完成无竞争时的快速路径，竞争时才陷入内核。PowerShell 的 `Monitor` 在功能上类似，但 `CRITICAL_SECTION` 可以通过 P/Invoke 在更底层使用。
**知识点**：`InitializeCriticalSection` 必须在任何 `Enter` 之前调用；`EnterCriticalSection` / `LeaveCriticalSection` 必须严格配对，通常用 `try/finally` 保证；`DeleteCriticalSection` 在不再使用后调用；`SRWLOCK` 的 `AcquireSRWLockExclusive` / `ReleaseSRWLockExclusive` 和共享版本。
**语法/类/函数**：`[DllImport("kernel32.dll")] public static extern void InitializeCriticalSection(out CRITICAL_SECTION cs);`；`[DllImport("kernel32.dll")] public static extern void EnterCriticalSection(ref CRITICAL_SECTION cs);`；`[DllImport("kernel32.dll")] public static extern void LeaveCriticalSection(ref CRITICAL_SECTION cs);`。

### 23.2 内核对象同步：`WaitForSingleObject` / `WaitForMultipleObjects` 与命名对象
**简介**：对标《Windows 核心编程》第9章“用内核对象进行线程同步”。Mutex、Event、Semaphore、Waitable Timer 都是可等待内核对象，`WaitForSingleObject` 等待单个对象，`WaitForMultipleObjects` 等待多个对象（“任一”或“全部”）。**命名内核对象**（如 `Global\MyMutex`）可以实现跨进程同步——这是 PowerShell 中协调多个脚本实例的底层机制。
**知识点**：`CreateMutex` / `CreateEvent` / `CreateSemaphore` 的 `lpName` 参数支持 `Global\` / `Local\` 前缀；`WaitForSingleObject` 的 `INFINITE`（`0xFFFFFFFF`）和超时毫秒；`WAIT_OBJECT_0` / `WAIT_TIMEOUT` / `WAIT_ABANDONED` 返回值；`ReleaseMutex` / `SetEvent` / `ReleaseSemaphore` 的配对操作。
**语法/类/函数**：`[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)] public static extern IntPtr CreateMutex(IntPtr attrs, bool initialOwner, string name);`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern uint WaitForSingleObject(IntPtr handle, uint milliseconds);`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern bool ReleaseMutex(IntPtr handle);`。

## 第24章 内存管理：虚拟内存、堆与内存映射文件

> **章内定位**：对应《Windows 核心编程》第III部分“内存管理”（第13–18章：Windows 内存架构、探索虚拟内存、使用虚拟内存、线程栈、内存映射文件、堆）。PowerShell 脚本通常不需要直接管理虚拟内存，但在**跨进程读取内存**、**大文件的内存映射处理**、**非托管缓冲区的手动分配**这些场景下，Win32 内存 API 是唯一路径。第20.4节的结构体封送已经涉及 `AllocHGlobal`，本章将其放入完整的虚拟内存语义中。

### 24.1 虚拟内存查询与提交：`VirtualQuery` / `VirtualAlloc`
**简介**：`VirtualQuery` 查询指定地址所在内存区域的状态（`MEM_COMMIT` / `MEM_RESERVE` / `MEM_FREE`）、保护属性（`PAGE_READWRITE` / `PAGE_EXECUTE_READ`）和类型（`MEM_IMAGE` / `MEM_MAPPED` / `MEM_PRIVATE`）。`VirtualAlloc` 在**当前进程**中保留或提交内存；`VirtualAllocEx` 在**目标进程**中分配（用于跨进程写入，参见第21章进程内存读写）。
**知识点**：`VirtualQuery` 填充 `MEMORY_BASIC_INFORMATION` 的 `BaseAddress` / `AllocationBase` / `RegionSize` / `State` / `Protect` / `Type`；`VirtualAlloc` 的 `MEM_RESERVE` 保留地址空间但不提交物理存储，`MEM_COMMIT` 提交；`VirtualFree` 的 `MEM_RELEASE`（释放保留区域）与 `MEM_DECOMMIT`（取消提交但保留地址）。
**语法/类/函数**：`[DllImport("kernel32.dll", SetLastError = true)] public static extern UIntPtr VirtualQuery(IntPtr addr, out MEMORY_BASIC_INFORMATION info, UIntPtr length);`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern IntPtr VirtualAlloc(IntPtr addr, UIntPtr size, uint allocType, uint protect);`。

### 24.2 内存映射文件：`CreateFileMapping` / `MapViewOfFile`
**简介**：对标《Windows 核心编程》第17章“内存映射文件”。内存映射文件将磁盘文件或页文件的一部分映射到进程地址空间，读写映射视图就是读写文件，无需 `ReadFile` / `WriteFile`。在 PowerShell 中，这一机制用于**大文件的高效随机访问**和**跨进程共享内存**（命名文件映射）。.NET 的 `MemoryMappedFile` 是更安全的托管替代，但 Win32 路径提供了更细粒度的保护属性和视图控制。
**知识点**：`CreateFileMapping(INVALID_HANDLE_VALUE, ...)` 创建基于页文件的共享内存；`MapViewOfFile` 返回映射视图的基地址；`UnmapViewOfFile` 解除映射；`CloseHandle` 关闭映射句柄；命名映射对象的 `Global\` / `Local\` 命名空间。
**语法/类/函数**：`[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)] public static extern IntPtr CreateFileMapping(IntPtr file, IntPtr attrs, uint protect, uint maxHigh, uint maxLow, string name);`；`[DllImport("kernel32.dll", SetLastError = true)] public static extern IntPtr MapViewOfFile(IntPtr mapping, uint access, uint offsetHigh, uint offsetLow, UIntPtr bytes);`。

## 第25章 动态链接库：基础、高级技术与注入

> **章内定位**：对应《Windows 核心编程》第IV部分“动态链接库”（第19–22章：DLL 基础、DLL 高级技术、线程局部存储、DLL 注入和 API 拦截）。PowerShell 中 `Add-Type -Path` 和 `[Reflection.Assembly]::LoadFrom()` 已经覆盖了**托管 DLL** 的加载。本章关注的是**非托管 DLL**：`LoadLibrary` / `GetProcAddress` 的动态加载、DLL 导出函数的调用、以及 DLL 注入这一高风险操作的边界。

### 25.1 非托管 DLL 的动态加载：`LoadLibrary` / `GetProcAddress` / `FreeLibrary`
**简介**：`LoadLibrary` 将非托管 DLL 映射到当前进程并返回 `HMODULE`；`GetProcAddress` 按名称获取导出函数的地址；`FreeLibrary` 递减 DLL 的引用计数。PowerShell 的 `Add-Type` 在编译时绑定 `DllImport`，而 `LoadLibrary` / `GetProcAddress` 提供的是**运行时动态绑定**——适合调用名称在编写脚本时未知的导出函数，或加载路径在运行时决定的 DLL。
**知识点**：`LoadLibrary` 的搜索路径顺序（应用程序目录、系统目录、PATH 等）；`GetProcAddress` 返回的 `IntPtr` 需要转换为委托才能调用（`Marshal.GetDelegateForFunctionPointer()`）；`FreeLibrary` 与 `LoadLibrary` 的配对；`LoadLibraryEx` 的 `LOAD_LIBRARY_SEARCH_*` 标志对搜索路径的精确控制。
**语法/类/函数**：`[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)] public static extern IntPtr LoadLibrary(string path);`；`[DllImport("kernel32.dll", CharSet = CharSet.Ansi, SetLastError = true)] public static extern IntPtr GetProcAddress(IntPtr module, string procName);`；`[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($ptr, [type][MyDelegate])`。

### 25.2 DLL 注入与 API 拦截的边界
**简介**：对标《Windows 核心编程》第22章“DLL 注入和 API 拦截”。`CreateRemoteThread` + `LoadLibrary` 是经典的 DLL 注入路径；`SetWindowsHookEx` 是另一种基于钩子的注入机制。**在 PowerShell 脚本中实施 DLL 注入是极高风险且通常不必要的操作**：它需要目标进程的 `PROCESS_ALL_ACCESS` 权限，会触发安全软件告警，且 PowerShell 自身的运行空间模型使得注入后的回调管理极为困难。本章将这一主题放在“边界”语境下讲授，重点在于**理解注入机制以便防御和诊断**，而非鼓励在脚本中使用。
**知识点**：`CreateRemoteThread` 在目标进程中启动 `LoadLibrary` 的注入路径；`SetWindowsHookEx` 的全局钩子需要 DLL 被映射到所有 GUI 进程；`OpenProcess` 的 `PROCESS_CREATE_THREAD | PROCESS_VM_OPERATION | PROCESS_VM_WRITE` 权限组合；**PowerShell 边界**：如果脚本需要“扩展目标进程行为”，应优先考虑目标进程本身是否提供了插件/扩展机制，而非注入。
**语法/类/函数**：`[DllImport("kernel32.dll", SetLastError = true)] public static extern IntPtr CreateRemoteThread(IntPtr process, IntPtr attrs, uint stackSize, IntPtr startAddr, IntPtr param, uint flags, out uint threadId);`；`[DllImport("user32.dll", SetLastError = true)] public static extern IntPtr SetWindowsHookEx(int hookType, IntPtr hookProc, IntPtr module, uint threadId);`。

## 第26章 结构化异常处理与错误报告

> **章内定位**：对应《Windows 核心编程》第V部分“结构化异常处理”（第23–26章：终止处理程序、异常处理程序与软件异常、未处理异常/向量化异常处理、错误报告与应用程序恢复）。PowerShell 的 `try/catch/finally` 和 `trap` 已经在语言层覆盖了托管异常处理（参见第8章）。本章关注的是**Windows 结构化异常处理（SEH）的底层机制**：`__try` / `__except` / `__finally` 在非托管代码中的行为、向量化异常处理（VEH）、以及 Windows 错误报告（WER）的触发条件。在 PowerShell 中，SEH 主要通过 .NET 的托管异常封装间接可见，理解 SEH 有助于诊断“PowerShell 进程崩溃”或“本机 DLL 抛出异常”这类跨边界故障。
**知识点**：SEH 的 `__try` / `__except` / `__finally` 与 C++ 异常和 .NET 托管异常的层次关系；`AddVectoredExceptionHandler` 作为进程级异常拦截的最后一道防线；`SetUnhandledExceptionFilter` 与 WER 的交互；**PowerShell 边界**：脚本层无法直接捕获非托管 SEH 异常；如果 P/Invoke 调用的本机函数抛出 SEH 异常，通常会绕过 PowerShell 的 `try/catch` 导致进程终止，因此对不可信本机 DLL 的调用必须隔离在独立进程中。
**语法/类/函数**：`[DllImport("kernel32.dll")] public static extern IntPtr AddVectoredExceptionHandler(uint first, IntPtr handler);`；`[DllImport("kernel32.dll")] public static extern IntPtr SetUnhandledExceptionFilter(IntPtr filter);`；托管侧 `AppDomain.CurrentDomain.UnhandledException` 事件作为 PowerShell 进程级兜底。

## 第27章 窗口、消息与输入

> **章内定位**：进入《Windows程序设计》的“窗口/消息/输入”主线。对应王艳平版第3章“Win32程序的执行单元”（消息循环）、第6章“框架中的窗口”、第7章“用户界面设计”，以及 Petzold 第5版基础知识部分的窗口与消息、键盘、鼠标章节。第19章已经用 WinForms 封装了 UI 构建；本章回到 **Win32 消息机制本身**：窗口句柄的查找、消息的发送与投递、输入模拟。PowerShell 脚本与已有窗口交互的最高频用途集中于此。

### 27.1 窗口查找：`FindWindow` / `FindWindowEx` / `EnumWindows`
**简介**：`FindWindow(lpClassName, lpWindowName)` 按类名和/或标题查找顶层窗口；`FindWindowEx` 在父窗口下查找子控件（如对话框按钮）。`EnumWindows` 通过回调枚举所有顶层窗口，适合按条件扫描。PowerShell 中的典型模式是：`FindWindow` 拿到 `HWND` → 后续 `SendMessage` 或 `ShowWindow` 操作。
**知识点**：`lpClassName` 或 `lpWindowName` 传 `$null` 表示匹配任意值；窗口类名可用 `GetClassName` 或 Spy++ 获取；`EnumWindows` 的回调需要 `Add-Type` 定义 `delegate` 并配合 `Marshal.GetFunctionPointerForDelegate()`；`IsWindow` 验证句柄有效性。
**语法/类/函数**：`[DllImport("user32.dll", SetLastError = true, CharSet = CharSet.Auto)] public static extern IntPtr FindWindow(string className, string windowName);`；`[DllImport("user32.dll", SetLastError = true, CharSet = CharSet.Auto)] public static extern IntPtr FindWindowEx(IntPtr parent, IntPtr childAfter, string className, string windowName);`。

### 27.2 窗口状态与控制：`ShowWindow` / `SetForegroundWindow` / `IsWindowVisible`
**简介**：`ShowWindow(hWnd, nCmdShow)` 控制显示状态（`SW_HIDE` / `SW_SHOW` / `SW_MINIMIZE` / `SW_RESTORE`）；`SetForegroundWindow` 前置窗口；`IsWindowVisible` 检查可见性。这三个函数构成“窗口自动化”的最小工具集。
**知识点**：`ShowWindowAsync` 与 `ShowWindow` 的区别——前者不等待窗口处理消息，跨进程时更安全；`SetForegroundWindow` 受前台锁限制，可能静默失败；`IsWindowVisible` 对最小化窗口返回 `true`，对隐藏窗口返回 `false`；`GetWindowRect` / `GetClientRect` 获取窗口和客户区坐标。
**语法/类/函数**：`[DllImport("user32.dll")] public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);`；`[DllImport("user32.dll")] public static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);`；`[DllImport("user32.dll")] public static extern bool SetForegroundWindow(IntPtr hWnd);`。

### 27.3 消息发送：`SendMessage` / `PostMessage` 与输入模拟
**简介**：`SendMessage` 同步发送消息并等待处理完成；`PostMessage` 将消息放入队列后立即返回。PowerShell 自动化中，`SendMessage` 用于 `WM_SETTEXT`（设置文本）、`BM_CLICK`（模拟按钮点击）、`WM_CLOSE`（关闭窗口）。`SendInput` 或 `keybd_event` / `mouse_event` 用于更底层的输入模拟，但受 UIPI 和 UAC 限制。
**知识点**：`WM_SETTEXT` 的 `lParam` 是字符串指针；`BM_CLICK` 的 `wParam` / `lParam` 均为 0；`WM_CLOSE` 发送关闭请求，`WM_DESTROY` 是实际销毁；**PowerShell 边界**：低完整性级别进程无法向高完整性级别窗口 `SendMessage`（UIPI 拦截）；跨会话（服务会话 vs 交互式桌面）的窗口操作会被会话隔离阻止。
**语法/类/函数**：`[DllImport("user32.dll", CharSet = CharSet.Auto)] public static extern IntPtr SendMessage(IntPtr hWnd, uint msg, IntPtr wParam, IntPtr lParam);`；`[DllImport("user32.dll", SetLastError = true)] public static extern uint SendInput(uint count, INPUT[] inputs, int size);`。

## 第28章 图形与文本输出：GDI 与设备上下文

> **章内定位**：对应《Windows程序设计》的“图形”部分（Petzold 第5版第13–18章：GDI 绘图、位图、调色板；王艳平版第4章“Windows图形界面”）。PowerShell 中图形输出的首选是 **WinForms 的 `System.Drawing`（GDI+ 托管封装）**，而不是裸 GDI32。本章的价值在于：理解 `HDC`（设备上下文）、GDI 对象（画笔/画刷/字体）和双缓冲的底层机制，以便在 `System.Drawing` 不够用时（如需要更细粒度的 GDI 控制或与遗留本机绘制代码交互）能够下沉到 `gdi32.dll`。

### 28.1 设备上下文：`GetDC` / `CreateCompatibleDC` 与 `BitBlt`
**简介**：`HDC` 是 GDI 绘制的“画布”句柄。`GetDC(hWnd)` 获取窗口客户区的设备上下文；`CreateCompatibleDC` 创建与目标 DC 兼容的内存 DC；`BitBlt` 在 DC 之间位块传输。PowerShell 中的典型用途是**窗口截图**：`GetDC` → `CreateCompatibleDC` / `CreateCompatibleBitmap` → `BitBlt` → `GetDIBits` 转回托管位图。
**知识点**：`GetDC` / `ReleaseDC` 的配对；`CreateCompatibleDC` / `DeleteDC` 的配对；`CreateCompatibleBitmap` / `DeleteObject` 的配对；`BitBlt` 的 `SRCCOPY` 光栅操作码；`GetDIBits` 将 GDI 位图转为 `byte[]`。
**语法/类/函数**：`[DllImport("user32.dll")] public static extern IntPtr GetDC(IntPtr hWnd);`；`[DllImport("gdi32.dll")] public static extern IntPtr CreateCompatibleDC(IntPtr hdc);`；`[DllImport("gdi32.dll")] public static extern bool BitBlt(IntPtr dest, int x, int y, int w, int h, IntPtr src, int srcX, int srcY, uint rop);`。

### 28.2 GDI 对象与文本输出：`CreatePen` / `CreateSolidBrush` / `TextOut`
**简介**：GDI 绘制依赖**GDI 对象**：`HPEN`（画笔）、`HBRUSH`（画刷）、`HFONT`（字体）。`SelectObject` 将对象选入 DC，绘制完成后应选回旧对象。`TextOut` / `DrawText` 在 DC 上输出文本。PowerShell 中这些 API 的实用性低于 `System.Drawing` 的 `Graphics.DrawString()`，但在需要**精确控制 GDI 字体度量**或**兼容遗留 GDI 调用**时仍有价值。
**知识点**：`CreatePen` 的样式/宽度/颜色参数；`CreateSolidBrush` 的颜色格式 `0x00BBGGRR`；`SelectObject` 返回旧对象，必须恢复；`DeleteObject` 删除不再使用的 GDI 对象；`GetTextExtentPoint32` 获取文本尺寸。
**语法/类/函数**：`[DllImport("gdi32.dll")] public static extern IntPtr CreateSolidBrush(uint color);`；`[DllImport("gdi32.dll")] public static extern IntPtr SelectObject(IntPtr hdc, IntPtr obj);`；`[DllImport("gdi32.dll", CharSet = CharSet.Auto)] public static extern bool TextOut(IntPtr hdc, int x, int y, string text, int length);`。

## 第29章 使用边界：PowerShell 中 Win32 调用的工程判断

> **章内定位**：本部分的收束章，呼应第5部分“什么时候不该用 PowerShell”的工程判断。Win32 API 在 PowerShell 中的代价不只是“多写几行 `Add-Type`”：会话级类型不可重定义、手动内存/句柄管理、封送错误、UIPI 与会话隔离、以及 32/64 位句柄宽度差异。本章给出**四条判断信号**和**安全清单**，将前九章的机制知识收束为可操作的工程决策。

### 29.1 优先使用 .NET / cmdlet 的判据
**简介**：大多数 Win32 API 的“常见用途”已经被 .NET 基类库或 PowerShell cmdlet 覆盖。判断是否应该下沉到 Win32 API 的核心线索是：**.NET 是否暴露了等价能力，且没有功能缺口**。如果等价能力存在且足够，Win32 路径只会增加复杂性和故障面。
**知识点**：进程/线程/同步/内存/窗口的 .NET 与 Win32 对照：`[System.Diagnostics.Process]` vs `CreateProcess`，`Monitor` / `SemaphoreSlim` vs `CRITICAL_SECTION` / `WaitForSingleObject`，`MemoryMappedFile` vs `CreateFileMapping`，`System.Drawing` vs GDI32；`GetVersionEx` 的兼容性问题使 `[System.Environment]::OSVersion` 成为更可靠的选择。
**语法/类/函数**：`[System.Diagnostics.Process]::Start()`；`[System.Threading.Monitor]::Enter()`；`[System.IO.MemoryMappedFiles.MemoryMappedFile]::CreateNew()`。

### 29.2 “必须用 Win32”的信号与安全清单
**简介**：有四类信号表明 Win32 API 是必要的：**通知机制**（`RegNotifyChangeKeyValue`）、**跨进程内存访问**（`ReadProcessMemory` / `VirtualAllocEx`）、**窗口消息交互**（`SendMessage` / `FindWindowEx`）、**令牌特权调整**（`AdjustTokenPrivileges`）。一旦决定调用，以下安全清单必须遵守，否则脚本会在不同机器、不同完整性级别或不同架构下间歇性失败。
**知识点**：**句柄释放**：`CloseHandle` / `ReleaseDC` / `DeleteDC` / `DeleteObject` 必须在 `finally` 中执行；**内存释放**：`AllocHGlobal` / `VirtualAlloc` 分配的缓冲区必须释放；**错误路径**：`SetLastError = true` + 立即 `GetLastWin32Error()`；**架构安全**：句柄用 `IntPtr` 而非 `[int]`；**完整性级别**：低完整性进程无法向高完整性窗口 `SendMessage`；**会话隔离**：服务会话中的脚本无法操作交互式桌面窗口；**非托管异常**：本机 DLL 抛出的 SEH 异常会绕过 PowerShell 的 `try/catch`，不可信调用应隔离在独立进程中。
**语法/类/函数**：`try { ... } finally { [Win32]::CloseHandle($h) }`；`$ptr = [Marshal]::AllocHGlobal($size); try { ... } finally { [Marshal]::FreeHGlobal($ptr) }`。


# 第5部分 工程化与互操作：从脚本到可交付工具

> **部分定位**：前四部分回答“怎么写”，本部分回答“怎么交付、怎么安全地跨越边界”。对标《C# 本质论》“部署/互操作性”主题和《CLR via C#》第24–25章“序列化/WinRT 互操作”的问题意识，但载体是 PowerShell 模块、脚本签名和 P/Invoke。

## 第30章 模块化：函数、脚本与模块

### 30.1 从脚本到函数到模块的演进路径
**简介**：一个 `.ps1` 脚本在规模增长后会经历“函数化 → 点源共享 → `.psm1` 模块化 → 清单文件”的自然演进，每一步都解决前一阶段的可复用性或可分发性问题。
**知识点**：`.psm1` 模块文件的自动加载语义；`Export-ModuleMember` 控制公开接口；`.psd1` 模块清单的 `RootModule`、`FunctionsToExport`、`RequiredModules`；模块作用域与全局作用域的隔离。
**语法/类/函数**：`New-ModuleManifest`、`Import-Module -Force`、`Get-Module -ListAvailable`、`$env:PSModulePath`。

### 30.2 模块设计原则：公开接口与内部实现的边界
**简介**：PowerShell 模块没有 C# 的 `public` / `internal` 关键字，公开与私有的边界完全由 `Export-ModuleMember` 和清单文件划定。
**知识点**：默认情况下函数在模块内可见但不导出；`Export-ModuleMember -Function` / `-Variable` / `-Alias`；嵌套模块与 `ScriptsToProcess`；模块卸载与 `Remove-Module` 的清理责任；第6章定义的 PowerShell 类在模块中默认不导出，需 `using module` 显式导入。
**语法/类/函数**：`Export-ModuleMember -Function 'Get-*' -Alias 'gth'`；`$ExecutionContext.SessionState.Module`；`using module MyModule`。

## 第31章 脚本安全与执行策略

### 31.1 执行策略、签名与信任
**简介**：PowerShell 的执行策略不是安全边界，而是“防止意外执行”的用户体验层；真正的代码信任依靠 Authenticode 签名。
**知识点**：`Restricted` / `RemoteSigned` / `AllSigned` / `Unrestricted` / `Bypass` 的语义；`Set-AuthenticodeSignature` 与 `Get-AuthenticodeSignature`；`Unblock-File` 解除下载标记；组策略对执行策略的优先覆盖。
**语法/类/函数**：`Get-ExecutionPolicy -List`、`Set-ExecutionPolicy -Scope CurrentUser`、`[System.Management.Automation.Signature]`。

## 第32章 P/Invoke 与 COM 互操作

### 32.1 通过 `Add-Type -MemberDefinition` 调用 Win32 API
**简介**：PowerShell 可以直接通过 `Add-Type` 嵌入带 `DllImport` 的 C# 签名，当场编译并调用非托管函数，从而触及 .NET 基类库未封装的部分。本章是第4部分所有 P/Invoke 调用的工程化收束：把“能调用”变成“可交付”。
**知识点**：`-MemberDefinition` 生成模板代码的机制；`DllImport` 的 `CharSet` / `CallingConvention` / `SetLastError`；`Add-Type` 类型不可重定义的会话限制；封送处理（marshaling）中字符串和句柄的常见陷阱；将 P/Invoke 声明组织为可复用的模块函数（如 `function Invoke-Win32Api`）。
**语法/类/函数**：`Add-Type -MemberDefinition '[DllImport("user32.dll")] public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);' -Name Win32 -Namespace Native`；`[Native.Win32]::ShowWindow()`。

### 32.2 COM 对象操作与释放
**简介**：`New-Object -ComObject` 可以自动化 Excel、WScript.Shell 等经典 COM 组件，但 RCW 的引用计数管理不当会导致进程残留——这是 PowerShell COM 互操作中最常见的生产事故。
**知识点**：RCW（Runtime Callable Wrapper）与引用计数；中间对象（如 `$excel.Workbooks`）产生的不可见 RCW 残留；`[System.Runtime.InteropServices.Marshal]::ReleaseComObject()` 与 `FinalReleaseComObject()`；`GC.Collect()` / `WaitForPendingFinalizers()` 的配合；无人值守场景下 Office COM 自动化的不支持性。
**语法/类/函数**：`[System.Runtime.InteropServices.Marshal]::ReleaseComObject($obj)`、`[System.GC]::Collect()`、`[System.GC]::WaitForPendingFinalizers()`。

## 第33章 与 C# 的协作边界：什么时候不该用 PowerShell

### 33.1 脚本的边界与“下沉到 C#”的信号
**简介**：对标《CLR via C#》和《C# 本质论》的工程判断：当类型安全、性能关键路径、复杂泛型算法或强编译期检查成为主导需求时，PowerShell 的“运行时绑定”优势会转为劣势，应当把核心逻辑下沉到 C# 程序集，PowerShell 只保留编排层。
**知识点**：PowerShell 在热路径上的解释开销与参数绑定开销；`Add-Type` 动态编译的启动成本；强类型 API 契约的可发现性与脚本的 `Get-Member` 可发现性之间的取舍；第6章“PowerShell 类无法实现真正封装”的工程后果；第29章“Win32 调用边界”的工程后果；何时选择“C# 类库 + PowerShell 模块封装”的混合架构。
**语法/类/函数**：`Add-Type -Path .\Core.dll` 加载预编译程序集；`[Reflection.Assembly]::LoadFrom()`；`using module` 语句（PowerShell 5+）静态导入模块中的类。


# 附：全书对标关系速查

| 本书部分/章 | 对标《C# 本质论》 | 对标《CLR via C#》 | 对标 Win32 著作 | PowerShell 独特扩展/边界 |
|---|---|---|---|---|
| 第1部分 第2章 类型基础 | 第2–3章 数据类型 | 第4–5章 类型基础/基元类型 | — | PSObject + ETS |
| 第1部分 第3章 变量与运算符 | 第1–3章 语法基础 | — | — | 集合语义的比较运算符 |
| 第1部分 第4章 流程控制 | 第4章 控制流 | — | — | `switch` 多模式匹配 |
| 第1部分 第5章 函数与脚本块 | 第6–8章 方法/参数 | 第8–9章 方法/参数 | — | 高级函数 + 延迟绑定 |
| **第1部分 第6章 面向对象编程** | **第5–9章 类与OOP** | **第6章 类型和成员基础** | — | **`hidden` 非 private；无自动属性；无 `virtual`/`override`；接口实现无编译期检查；`NoRunspaceAffinity`** |
| 第1部分 第7章 泛型与 LINQ 风格 | 第12、15–16章 | 第12章 泛型 | — | 管道作为查询执行引擎 |
| 第1部分 第8章 异常处理 | 第10章 | 第20章 | — | 终止性/非终止性双通道 |
| 第2部分 第9–12章 管道与命令 | — | — | — | 对象流、参数绑定、格式化 |
| 第3部分 第13章 程序集与反射 | 第18–19章 | 第2–3、23章 | — | `Add-Type` 动态类型 |
| 第3部分 第14章 网络 | — | 异步 I/O / 程序集宿主 | — | cmdlet 与 `HttpClient`/Socket 的分工 |
| 第3部分 第15章 文本与正则 | 第3章 字符串 | 第14章 | — | `-match` 大小写不敏感；`-replace` 的 `$` 插值冲突 |
| 第3部分 第16章 容器 | 第12章 泛型集合 | 第12章 泛型 | — | `$using:` 只读快照；并发集合传引用 |
| 第3部分 第17章 线程与并发 | 第20章 异步/线程 | 第26–30章 | — | 四层并发栈选型；运行空间线程关联性 |
| 第3部分 第18章 IO 深化 | — | 第14章 流与字符适配 | — | `File.ReadLines()` 延迟枚举；管道与流的模型对立 |
| 第3部分 第19章 WinForms | — | STA / 消息泵 | — | PowerShell 7 默认 MTA，须 `pwsh -STA` |
| **第4部分 第20章 基础必备知识** | — | — | **《Windows核心编程》第1–3章** | `SetLastError` + `CharSet` + `IntPtr` 句柄管理 |
| **第4部分 第21章 进程与作业** | — | — | **《Windows核心编程》第4–5章** | `Start-Process` 不够用时才下沉 |
| **第4部分 第22章 线程基础与调度** | — | 第26章 线程基础 | **《Windows核心编程》第6–7章** | `Start-ThreadJob` 更安全；Win32 用于纯本机回调 |
| **第4部分 第23章 线程同步** | — | 第26章 同步构造 | **《Windows核心编程》第8–9章** | `Monitor` 够用；命名内核对象用于跨进程 |
| **第4部分 第24章 内存管理** | — | — | **《Windows核心编程》第13–18章** | `MemoryMappedFile` 优先；跨进程用 `VirtualAllocEx` |
| **第4部分 第25章 动态链接库** | 第19章 程序集 | 第23章 程序集加载 | **《Windows核心编程》第19–22章** | `Add-Type` 管托管；非托管动态绑定才用 `LoadLibrary` |
| **第4部分 第26章 结构化异常** | 第10章 异常 | 第20章 异常 | **《Windows核心编程》第23–26章** | SEH 绕过 `try/catch`；不可信 DLL 隔离进程 |
| **第4部分 第27章 窗口、消息与输入** | — | — | **《Windows程序设计》窗口/消息/输入** | 与已有窗口交互的高频区；UIPI 限制 |
| **第4部分 第28章 图形与文本输出** | — | — | **《Windows程序设计》GDI 部分** | `System.Drawing` 优先；GDI 用于截图/遗留兼容 |
| **第4部分 第29章 使用边界** | 全书工程判断 | 全书运行时视角 | 全书系统机制 | 四条信号 + 句柄/内存/错误/完整性清单 |
| 第5部分 第30章 模块化 | 第19章 程序集部署 | 第2–3章 程序集打包 | — | `.psd1` 清单与导出边界 |
| 第5部分 第31章 脚本安全 | — | — | — | 执行策略非安全边界；Authenticode 签名 |
| 第5部分 第32章 P/Invoke 与 COM | 第21章 互操作性 | 第25章 WinRT 互操作 | 第4部分全部 | 脚本级动态编译；RCW 释放 |
| 第5部分 第33章 协作边界 | 全书工程判断 | 全书运行时视角 | 全书系统机制 | “C# 类库 + PowerShell 编排”混合架构 |


**细纲使用建议**：每一节的“知识点”可直接作为该节的写作检查清单——如果某节缺少其中一项，说明该节没有完成“语法 + 命令/机制 + .NET 设施”三者之间的闭环。第3部分的每一章尤其需要保持“CLR 事实—PowerShell 暴露方式—调用边界”的三段式写法；其中网络、文本/正则、WinForms 是 PowerShell 特有陷阱最密集的区域，IO 与容器模块的关键在于性能与并发正确性，线程模块应始终以“PowerShell 四层并发栈如何选型”为叙述主线。第4部分的章节顺序本身就是一条“从底层机制到上层界面”的认知路径：第20章必须放在最前，因为错误码、字符串封送和句柄管理是后面所有章节的共同语法；第21–26章遵循《Windows 核心编程》的“进程→线程→同步→内存→DLL→异常”机制链，每一章都要在第3部分已有 .NET 知识的基础上做“Win32 还剩下什么”的判断；第27–28章转入《Windows程序设计》的界面主线，但始终提醒读者：PowerShell 中 WinForms / System.Drawing 是首选，裸 Win32 窗口消息和 GDI 只在“与已有窗口交互”或“托管封装不够用”时才有价值；第29章将九章的机制知识收束为工程决策，与第5部分的“什么时候不该用 PowerShell”形成前后呼应。