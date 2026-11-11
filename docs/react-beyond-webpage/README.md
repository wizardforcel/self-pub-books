# React 不止于网页：全生态开发实战

> 从组件到移动端、视频与游戏 —— 面向 JavaScript 入门者的 React 全栈实战书籍

这是一本以 **React 19** 为核心、覆盖前端全生态的实战型书籍。书名《React 不止于网页》的意思是：React 不仅能做传统的网页开发，还能用来构建移动端应用、制作视频、开发游戏，甚至编写插件与工程化发布流程。

## 适合谁阅读

- 掌握 JavaScript 基础（变量、函数、数组、对象、箭头函数等）的编程入门者
- **没有学过任何其他编程语言**，也没接触过 React
- 想从零开始，系统学习 React 及其周边生态，并通过真实项目上手

全书语言详细且直白，重要概念用大白话解释，每一节都附带可运行的代码示例，核心概念配有 Mermaid 图表。

## 目录概览

| 章节 | 内容 |
|------|------|
| 第1章 | React 核心组件与渲染模型（CRA→Vite、函数组件、JSX、Props、key、受控/非受控） |
| 第2章 | 状态管理：客户端状态与服务端状态的分离（useState/useReducer、Context、Zustand、Redux Toolkit、Jotai） |
| 第3章 | React Hooks 深度与自定义 Hook 设计（useEffect 替代思维、useRef、自定义 Hook、Hook 集合库） |
| 第4章 | 路由：React Router v7 数据模式（createBrowserRouter、loader、action、Form、useFetcher、懒加载、受保护路由） |
| 第5章 | 网络与数据层：TanStack Query + HTTP 客户端（useQuery、useMutation、Axios/ky/fetch、集中式 API 模块） |
| 第6章 | 样式与组件库：Bootstrap 与 Ant Design（react-bootstrap、antd、两者共存与边界） |
| 第7章 | 测试策略：从组件到端到端（Vitest、React Testing Library、MSW、Playwright、覆盖率边界） |
| 第8章 | 项目实战一：单页面应用（SPA）—— 个人任务管理应用 TaskFlow |
| 第9章 | 项目实战二：多页面应用 —— 电商后台管理系统 ShopAdmin |
| 第10章 | React Native 移动端（Expo、核心组件、Expo Router、NativeWind、状态与数据层、移动端测试） |
| 第11章 | 项目实战三：移动端应用 —— 习惯打卡应用 HabitLoop |
| 第12章 | Remotion：用 React 制作视频（帧模型、插值与弹簧动画、序列转场、媒体字幕与渲染输出） |
| 第13章 | React 游戏开发（PixiJS 2D、React Three Fiber 3D、游戏状态与 60FPS 优化、输入与移动端适配） |
| 第14章 | 项目实战四：游戏项目 —— 太空射击游戏 StarDrift |
| 第15章 | 插件、工程化与发布（React 插件化架构、Vite/TypeScript/ESLint 工具链、发布与性能基线、CI/CD） |

推荐线性阅读路径：`1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10 → 11 → 12 → 13 → 14 → 15`。若以移动端为目标，读完第9章后可直接进入第10–11章；第12–14章作为选读。四个项目均可独立运行，也可作为课程设计或毕业项目的参考实现。

## 四个实战项目

1. **TaskFlow（第8章，SPA）** —— 个人任务管理应用，单路由树、客户端状态为主、TanStack Query 接入服务端状态、Ant Design 样式，配齐组件测试与 E2E 测试。
2. **ShopAdmin（第9章，多页面应用）** —— 电商后台管理系统，嵌套路由、loader/action 数据流、权限守卫、代码分割、Ant Design 与 React-Bootstrap 分区共存。
3. **HabitLoop（第11章，移动端）** —— 习惯打卡应用，Expo Router 文件式导航、NativeWind 样式、Zustand 本地状态、TanStack Query 云端同步、本地通知与安全存储。
4. **StarDrift（第14章，游戏）** —— 太空射击游戏，PixiJS 渲染 2D 场景、Zustand 管理游戏状态、Howler.js 音效、键盘与触摸双端操作。

## 技术栈

- React 19 + Vite + TypeScript
- 状态管理：Zustand、Redux Toolkit、Jotai、Context
- 路由：React Router v7（Web）、Expo Router（移动端）
- 网络：TanStack Query、Axios、ky、fetch、MSW
- 样式：Ant Design、React-Bootstrap、NativeWind、Sass/CSS
- 测试：Vitest、React Testing Library、@testing-library/react-native、Playwright
- 移动端：Expo SDK、React Native、expo-secure-store、expo-notifications、EAS Build
- 视频：Remotion、@remotion/renderer
- 游戏：PixiJS + @pixi/react、React Three Fiber、Howler.js
- 工程化：ESLint flat config、GitHub Actions、lighthouse-ci、rollup-plugin-visualizer

## 章节文件

每一章对应仓库中的一个 Markdown 文件：

- `ch1.md` ~ `ch15.md` —— 书籍正文（按章节顺序）
- `asset/detail.md` —— 详细细纲（含每节的知识点、对应类/函数/库）
- `AGENTS.md` —— 写作要求与项目约定

## 写作约定（来自 AGENTS.md）

1. 编写每章正文之前先调用百度搜索，返回至少 20 条结果
2. 面向读者是掌握 JS 的编程入门者，没有学过任何其它编程语言
3. 语言风格详细且直白，重要概念用大白话解释，可以添加一些废话，使行文更加自然流畅
4. 每一节都附带代码示例，解释类或函数等库的设施
5. 核心概念添加 Mermaid 图表来解释
6. 细纲位于 `asset/detail.md`

## 本地阅读

直接在浏览器或 Markdown 编辑器中打开 `ch1.md` ~ `ch15.md` 即可阅读各章正文。建议按章节顺序阅读，并边读边运行每节中的代码示例。

---

*由 [wizardforcel](https://github.com/wizardforcel) 维护，Claude Code 协作编写。*
