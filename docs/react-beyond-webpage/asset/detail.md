# 《React 不止于网页：全生态开发实战，从组件到移动端、视频与游戏》图书细纲（含四个项目实战）

> 本版在上一版基础上插入四个实战项目章：**项目实战一（SPA）** 置于第7章测试之后、**项目实战二（多页面复杂路由）** 紧随其后、**项目实战三（移动端）** 置于移动端技术章之后、**项目实战四（游戏）** 置于游戏技术章之后。原第8–11章顺延为第10–15章。


## 第1章 React 核心组件与渲染模型

### 1.1 React 项目初始化：从 Create React App 到 Vite

本节用 CRA 生成 React 19 项目，理解"零配置脚手架"的历史价值与退役原因（CRA 已于 2025 年 2 月被官方正式弃用），随后**不重写业务代码**，只替换构建器，把同一项目迁移到 Vite 并在浏览器中验证运行效果；最后给出今天从零开始的标准命令。重点提示迁移中的三个坑：入口 `index.html` 从 `public/` 移到根目录、`%PUBLIC_URL%` 占位符删除、`REACT_APP_` 环境变量前缀改为 `VITE_` 且访问方式变为 `import.meta.env`。对应类/函数/库：`create-react-app`、`react-scripts`、`vite`、`@vitejs/plugin-react`、`defineConfig`、`import.meta.env`、`npm create vite@latest`。

### 1.2 函数组件与 JSX 运行时

函数组件是 React 的基元。本节从 **React 19 的自动 memoization（React Compiler）** 出发，解释函数组件为什么在 2026 年仍然是默认选择，以及编译器如何改变了"何时需要手写 memo"的判断逻辑。对应类/函数/库：`React.FC`、`JSX.Element`、React Compiler（`babel-plugin-react-compiler`）。

### 1.3 Props 设计与组件契约

Props 是组件之间唯一的公开接口。本节聚焦 Props 的**类型设计**与**运行时边界**：如何用 TypeScript 区分"配置型 Props"与"数据型 Props"，何时用 `children` 而非具名 Props，以及 `propTypes` 在 2026 年是否还有存在价值。对应类/函数/库：`React.ReactNode`、`React.PropsWithChildren`、`ComponentPropsWithoutRef`。

### 1.4 条件渲染、列表与 key 的深层语义

`key` 不是"消除警告的开关"，而是 React 协调算法的身份标识。本节通过具体反例说明 key 选择如何影响组件状态保留、动画触发和性能，并给出列表虚拟化场景下的 key 策略。对应类/函数/库：`Array.prototype.map`、`React.Fragment`（`<Fragment key={}>`）、`@tanstack/react-virtual`。

### 1.5 合成事件与受控/非受控模式

React 的事件系统已完全统一为**根容器委托**。本节解释 `onChange` 与原生 `input` 事件的行为差异、受控组件的重渲染代价，以及何时切换到非受控模式（`defaultValue` / `ref`）来绕开性能瓶颈。对应类/函数/库：`React.ChangeEvent`、`useRef`、`defaultValue`、`onChange`。


## 第2章 状态管理：客户端状态与服务端状态的分离

### 2.1 局部状态：useState 与 useReducer 的适用边界

`useState` 适合"单值、独立、简单更新"的场景；`useReducer` 适合"多字段联动、更新逻辑可命名"的场景。本节用可运行的对照示例给出判断标准，并说明为什么"不要急着把状态提升到全局"是状态管理的第一原则。对应类/函数/库：`useState`、`useReducer`、`Dispatch<Action>`。

### 2.2 Context 的正确用途与性能陷阱

Context 不是状态管理器，而是**依赖注入**机制。本节区分"低频变更的配置/主题/用户身份"与"高频变更的 UI 状态"，解释为什么后者放进 Context 会导致整棵子树重渲染，以及 `use(Context)` 在 React 19 中的读取语义。对应类/函数/库：`createContext`、`useContext` / `use`、`Context.Provider`。

### 2.3 Zustand：轻量外部 store 与选择器订阅

Zustand 用模块级闭包保存状态，组件通过 `useStore(selector)` 订阅切片，无需 Provider。本节覆盖 `create`、`persist` 中间件、`shallow` 比较，以及如何在 React 外部（如路由 loader 或工具函数中）直接调用 `store.getState()` / `setState()`。对应类/函数/库：`zustand` 的 `create`、`useStore`、`persist`、`shallow`。

### 2.4 Redux Toolkit：可审计的单一状态树

RTK 的定位已经清晰：**企业级审计与时间旅行调试的标杆**，而非默认起点。本节聚焦 `createSlice`、`configureStore` 和 `createSelector`，说明在什么规模与合规要求下 RTK 的样板成本是值得的。对应类/函数/库：`@reduxjs/toolkit` 的 `createSlice`、`configureStore`、`createSelector`、`useSelector`、`useDispatch`。

### 2.5 Jotai：原子化状态与细粒度渲染

Jotai 以 `atom` 为最小状态单位，派生 atom 通过 `get` 组合，依赖关系自动追踪。本节用"表单草稿 + 派生校验 + 异步 atom"的完整示例展示其"渲染爆炸半径最小"的优势，并讨论 atom 数量增长后的命名与分组约定。对应类/函数/库：`jotai` 的 `atom`、`useAtom`、`useAtomValue`、`useSetAtom`、`atomWithStorage`。


## 第3章 React Hooks 深度与自定义 Hook 设计

### 3.1 useEffect 的替代思维：事件驱动与派生状态

许多 `useEffect` 是不必要的：派生状态用渲染期计算，用户交互用事件处理器，外部同步用 `useSyncExternalStore`。本节建立一套"先问是否真的需要 Effect"的决策流程。对应类/函数/库：`useEffect`、`useSyncExternalStore`、`useMemo`。

### 3.2 useRef、useImperativeHandle 与命令式逃生舱

`ref` 不只是 DOM 引用。本节覆盖 `useRef` 存储可变值、`useImperativeHandle` 暴露受限的命令式 API，以及何时应该拒绝命令式接口而保持声明式契约。对应类/函数/库：`useRef`、React 19 中 ref 作为 prop 直接传递、`useImperativeHandle`。

### 3.3 自定义 Hook 的设计原则：组合、命名与副作用隔离

自定义 Hook 的价值不在于"复用代码"，而在于**复用状态逻辑的契约**。本节以 `useDebounce`、`useLocalStorage`、`useAsync` 为例，讨论"一个 Hook 只做一件事"、返回值形态（元组 vs 对象）的选择，以及副作用清理的边界。对应类/函数/库：`useDebounce`、`useLocalStorage`、`useAsync`（手写实现）、`usehooks-ts`。

### 3.4 生产级 Hook 集合库的取舍：Rooks、ReactUse、ahooks

直接使用成熟的 Hook 集合库可以避免重复造轮子，但也会引入不必要的包体积和行为假设。本节对比 Rooks（125+ hooks，tree-shakeable，TypeScript-first）、ReactUse（110+）和 ahooks 的定位差异，给出"手写 vs 引入"的判断框架。对应类/函数/库：`rooks`、`react-use`、`ahooks`、`@reactuses/core`。


## 第4章 路由：React Router v7 数据模式

### 4.1 为什么必须使用 Data Mode：createBrowserRouter 与 RouterProvider

React Router v7 的 Data Mode 以 `createBrowserRouter` + `<RouterProvider>` 为核心，`loader`、`action`、`errorElement` 只在此模式下生效。`<BrowserRouter>` + `<Routes>` 的声明式模式会**静默忽略** `loader` prop，这是最常见的路由迁移陷阱。对应类/函数/库：`createBrowserRouter`、`RouterProvider`（从 `react-router/dom` 导入）、`RouteObject`。

### 4.2 loader：渲染前数据获取与并行加载

`loader` 在路由组件渲染之前运行，返回普通对象（v7 已移除 `json()` 和 `defer()`）。本节覆盖根路由 `loader` 的全局数据、嵌套路由的并行 `loader`、以及 `useLoaderData` 的类型化读取。对应类/函数/库：`loader`、`useLoaderData`、`LoaderFunctionArgs`。

### 4.3 action、Form 与 useFetcher：变更与乐观 UI

`action` 处理非 GET 请求，`<Form method="POST">` 提交后自动触发 `action` 并重新验证 `loader`。`useFetcher` 允许在不导航的情况下加载或提交数据，是实现乐观 UI 的关键工具。对应类/函数/库：`action`、`<Form>`、`useFetcher`、`useNavigation`（`state === "submitting"`）。

### 4.4 懒加载、错误边界与受保护路由

`route.lazy` 实现路由级代码分割；根路由的 `errorElement` 是错误冒泡的最终捕获点，缺失它会导致单个 `loader` 失败替换整个应用。受保护路由通过无路径布局路由 + `loader` 中的 `throw redirect()` 实现。对应类/函数/库：`route.lazy`、`errorElement`、`useRouteError`、`isRouteErrorResponse`、`redirect`、`Outlet`。


## 第5章 网络与数据层：TanStack Query + HTTP 客户端

### 5.1 服务端状态不是客户端状态：为什么需要 TanStack Query

架构共识是：**服务端状态（接口数据、缓存、轮询、失效重试）交给 TanStack Query，客户端状态才进入 Zustand/Redux/Jotai**。本节解释 `staleTime`、`gcTime`、`retry` 的语义，以及 `useQuery` 如何消除手写 `useEffect` 数据获取的竞态与加载状态管理。对应类/函数/库：`@tanstack/react-query` 的 `useQuery`、`QueryClient`、`QueryClientProvider`。

### 5.2 useMutation 与缓存失效：变更后的正确收尾

`useMutation` 处理 POST/PUT/DELETE，`onSuccess` 中调用 `queryClient.invalidateQueries` 触发相关查询重新获取。本节覆盖乐观更新（`onMutate` + 回滚）和错误处理的标准模式。对应类/函数/库：`useMutation`、`queryClient.invalidateQueries`、`queryClient.setQueryData`。

### 5.3 HTTP 客户端选型：Axios、ky 与 fetch

`axios` 提供拦截器、自动 JSON 转换和取消请求，是"熟悉的可靠选择"；`ky` 是更小的 fetch 封装；原生 `fetch` 已足够用于简单场景。本节给出三者的选型矩阵，并演示如何将 Axios 实例注入 TanStack Query 的 `queryFn`。对应类/函数/库：`axios`（`axios.create`、`interceptors`）、`ky`、`AbortController`。

### 5.4 数据层架构：从组件散落到集中式 API 模块

将 API 调用收敛到 `lib/api.ts` 或 `api/` 目录，组件只消费 Hook，不直接接触 HTTP 客户端。本节展示"API 函数 → Query Hook → 组件"的三层结构，以及在 React Native 中复用同一套数据层的做法。对应类/函数/库：自定义 `useQuery` 封装、`queryOptions`（TanStack Query 的类型安全模式）。


## 第6章 样式与组件库：Bootstrap 与 Ant Design

### 6.1 React-Bootstrap：工具类优先的轻量接入

`react-bootstrap` 将 Bootstrap 的 CSS 类映射为 React 组件，定制通过 Sass 变量或 CSS 覆盖完成，**没有 JavaScript 主题引擎**。它适合内容型页面和对 Bootstrap 网格/工具类已有依赖的项目。对应类/函数/库：`react-bootstrap` 的 `Button`、`Table`、`Container`、`Row`、`Col`。

### 6.2 Ant Design：企业级数据密集型 UI

`antd` 的定位是**数据密集型后台与管理界面**，`Table`、`Form`、`DatePicker` 等组件的完整度远超 Bootstrap 生态。主题定制通过 `ConfigProvider` 的 `theme.token` 完成。本节覆盖 `Table` 的排序/筛选/分页、`Form` 的受控校验集成，以及 `ConfigProvider` 的全局 token 管理。对应类/函数/库：`antd` 的 `ConfigProvider`、`Table`、`Form`、`Form.Item`、`DatePicker`、`theme`。

### 6.3 两者共存与边界划分

同一项目中可以并存 React-Bootstrap 与 antd，但需要明确边界：用 antd 处理后台数据界面，用 React-Bootstrap 处理面向用户的轻量页面，避免两套设计语言在同一屏内混用。本节给出目录隔离与主题变量协调的具体方案。对应类/函数/库：CSS 变量桥接、`ConfigProvider` 的 `token` 与 Bootstrap Sass 变量对照。


## 第7章 测试策略：从组件到端到端

### 7.1 测试分层与工具选型

测试不是"一个工具打天下"，而是分层：单元/组件测试用 Vitest + React Testing Library，API 层用 MSW 拦截，端到端用 Playwright。本节给出分层决策表，说明每层该测什么、不该测什么。对应类/函数/库：`vitest`、`@testing-library/react`、`msw`、`playwright`。

### 7.2 组件测试：以用户行为为断言中心

Testing Library 的核心原则是"测试用户能看到和做到的事"，而非组件内部实现。本节覆盖 `render`、`screen.getByRole`、`userEvent.click` 的标准写法，以及为什么应优先用 role/label 查询而非 `data-testid`。对应类/函数/库：`render`、`screen`、`getByRole`、`@testing-library/user-event`、`@testing-library/jest-dom`。

### 7.3 Hook 测试：renderHook 与副作用清理

自定义 Hook 需要独立测试其状态流转与清理逻辑。本节用 `useDebounce`、`useAsync` 演示 `renderHook` + `act` + `waitFor` 的写法，并强调清理函数（`clearTimeout`、`AbortController`）必须被断言覆盖。对应类/函数/库：`renderHook`、`act`、`waitFor`、`vi.useFakeTimers`。

### 7.4 路由与数据层集成测试：MSW 与 loader/action

路由的 `loader`/`action` 依赖网络，直接打真实接口会让测试不稳定。本节用 MSW 在 Node 侧拦截请求，配合 `createMemoryRouter` 测试 loader 返回、错误边界和重定向。对应类/函数/库：`msw` 的 `http`、`HttpResponse`、`setupServer`、`createMemoryRouter`、`RouterProvider`。

### 7.5 端到端测试：Playwright 与关键用户旅程

E2E 只覆盖"关键旅程"（登录、下单、发布），不追求覆盖率。本节覆盖 Playwright 的 `page.goto`、`getByRole`、`expect(...).toBeVisible()`、trace 回放，以及 CI 中的并行与重试配置。对应类/函数/库：`@playwright/test` 的 `test`、`expect`、`page`、`playwright.config.ts`。

### 7.6 覆盖率的合理边界

覆盖率是信号而非目标。本节讨论行覆盖 vs 分支覆盖、哪些代码值得 100% 覆盖（状态机、金额计算）、哪些不值得（纯样式组件），以及如何把覆盖率门槛写进 CI 而不拖慢迭代。对应类/函数/库：`vitest --coverage`、`@vitest/coverage-v8`、CI 阈值配置。


## 第8章 项目实战一：单页面应用（SPA）

> 本项目把第1–7章的知识收束为一个可上线的单页面应用。选题为**个人任务管理应用（TaskFlow）** ：单路由树、客户端状态为主、服务端状态通过 TanStack Query 接入、样式用 Ant Design，最终配齐组件测试与 E2E 测试。

### 8.1 需求分析与技术选型

明确 SPA 的功能边界（任务 CRUD、标签筛选、拖拽排序、本地持久化），并逐一对应技术选型：Vite + React 19 + TypeScript、Zustand 管客户端状态、TanStack Query 管服务端状态、Ant Design 提供表格与表单、Vitest/Playwright 负责测试。本节给出选型矩阵与"为什么不用 Redux"的论证。对应类/函数/库：`vite`、`zustand`、`@tanstack/react-query`、`antd`。

### 8.2 项目初始化与目录架构

从第1章的 Vite 初始化命令出发，搭建 `src/features/`、`src/components/`、`src/lib/`、`src/hooks/` 的分层目录，配置路径别名、ESLint flat config 与 TypeScript 严格模式。本节强调"按功能切分而非按文件类型切分"的目录原则。对应类/函数/库：`vite.config.ts` 的 `resolve.alias`、`tsconfig.json` 的 `paths`、`eslint.config.js`。

### 8.3 组件骨架与 Ant Design 布局

用 Ant Design 的 `Layout`、`Menu`、`Card`、`Table` 搭建应用外壳，用 `Form` 实现任务创建表单，用 `Modal` 实现编辑弹窗。本节演示如何把 `antd` 的受控表单与 Zustand store 对接，避免"表单状态双写"。对应类/函数/库：`Layout`、`Menu`、`Card`、`Table`、`Form`、`Form.Item`、`Modal`、`ConfigProvider`。

### 8.4 Zustand 状态设计与持久化

把任务列表、筛选条件、拖拽顺序拆成独立的 Zustand slice，用 `persist` 中间件写入 `localStorage`，用 `shallow` 比较避免无关重渲染。本节给出 store 文件结构与 selector 的命名约定。对应类/函数/库：`create`、`persist`、`shallow`、`useStore`。

### 8.5 TanStack Query 接入服务端数据

用 `useQuery` 拉取任务列表、`useMutation` 处理创建/更新/删除，`onSuccess` 中 `invalidateQueries` 刷新缓存；乐观更新用 `onMutate` + 回滚实现拖拽排序的即时反馈。本节同时演示 MSW 在开发环境模拟接口。对应类/函数/库：`useQuery`、`useMutation`、`queryClient.invalidateQueries`、`onMutate`、`msw`。

### 8.6 组件测试与 E2E 测试

为任务表单写 Testing Library 组件测试（提交、校验、错误提示），为"创建→筛选→完成→删除"主旅程写 Playwright E2E。本节复用第7章的分层策略，展示测试如何反过来约束组件 Props 设计。对应类/函数/库：`render`、`screen.getByRole`、`userEvent`、`@playwright/test`。

### 8.7 构建、性能检查与部署

执行 `vite build` 分析产物体积，用 `rollup-plugin-visualizer` 定位 Ant Design 的按需引入问题，配置 `manualChunks` 拆分第三方依赖，最终部署到静态托管平台。对应类/函数/库：`vite build`、`rollup-plugin-visualizer`、`manualChunks`、`web-vitals`。


## 第9章 项目实战二：多页面应用（复杂路由与架构）

> 本项目聚焦**多页面、多角色、多层级路由**的复杂应用。选题为**电商后台管理系统（ShopAdmin）** ：嵌套路由、loader/action 数据流、权限守卫、代码分割、React-Bootstrap 与 Ant Design 分区共存。

### 9.1 需求分析与路由架构设计

梳理商品管理、订单管理、用户管理、数据看板四大模块，画出路由树：根布局路由 → 登录页（无布局）→ 受保护布局路由 → 各业务模块嵌套路由。本节给出 `createBrowserRouter` 的完整 `RouteObject` 树形结构。对应类/函数/库：`createBrowserRouter`、`RouteObject`、`RouterProvider`。

### 9.2 嵌套布局与 Outlet 组合

用无路径布局路由承载侧边栏与顶栏，业务页面通过 `<Outlet>` 渲染；用 `useMatches` 驱动面包屑，用 `handle` 字段传递路由元信息。本节演示"布局路由 + 索引路由 + 动态路由"的组合模式。对应类/函数/库：`Outlet`、`useMatches`、`handle`、`index: true`。

### 9.3 loader/action 驱动的数据流

每个业务路由配置 `loader` 预取列表数据、`action` 处理表单提交并自动重新验证；用 `useNavigation` 展示全局提交状态，用 `useFetcher` 实现行内编辑而不触发导航。本节给出"列表页 loader + 详情页 loader + 编辑 action"的完整闭环。对应类/函数/库：`loader`、`action`、`useLoaderData`、`useNavigation`、`useFetcher`、`redirect`。

### 9.4 权限守卫与受保护路由

用根布局 `loader` 读取会话，未登录时 `throw redirect("/login")`；用角色字段在嵌套布局 `loader` 中做二级鉴权；登录 `action` 成功后重定向回原路径（`redirect(params.get("from"))`）。本节强调"鉴权在 loader 层完成，而非组件内 `useEffect`"。对应类/函数/库：`redirect`、`useRouteLoaderData`、`createCookieSessionStorage`（或 mock 会话层）。

### 9.5 代码分割与路由级懒加载

用 `route.lazy` 对每个业务模块做路由级代码分割，配合 `<Suspense>` 与加载骨架；对比 `React.lazy` 与 `route.lazy` 的适用差异。本节给出拆分前后的 bundle 对比数据。对应类/函数/库：`route.lazy`、`React.lazy`、`Suspense`、`rollup-plugin-visualizer`。

### 9.6 样式库分区共存：Ant Design + React-Bootstrap

后台数据界面用 antd（`Table`、`Form`、`DatePicker`），面向客户的商品预览页用 React-Bootstrap（`Container`、`Row`、`Col`、`Card`），通过目录隔离与 `ConfigProvider` token 桥接避免设计语言冲突。对应类/函数/库：`ConfigProvider`、`theme.token`、`react-bootstrap` 的 `Container`/`Row`/`Col`。

### 9.7 集成测试与 E2E 旅程

用 `createMemoryRouter` + MSW 测试 loader 鉴权重定向、action 提交后重新验证；用 Playwright 覆盖"登录→商品列表→编辑→保存→登出"完整旅程。对应类/函数/库：`createMemoryRouter`、`msw`、`@playwright/test`。


## 第10章 React Native 移动端

### 10.1 Expo 与项目初始化：移动端的标准起点

React Native 的推荐起点已是 **Expo + SDK 52+（新架构 Fabric/TurboModules 默认启用）**，`npx create-expo-app@latest` 是标准初始化命令。本节覆盖 Expo 项目结构、开发构建与生产构建的基本流程，并与第1章的 Vite 初始化做对照。对应类/函数/库：`create-expo-app`、`expo`、Expo SDK。

### 10.2 核心组件与布局：View、Text、Image、Pressable

移动端 UI 基元与 Web 不同：`View` 是 flex 容器（默认 `flexDirection: column`），`Text` 必须包裹所有文本，`Pressable` 替代 `div onClick`。本节建立"从 Web DOM 到 RN 组件"的映射表。对应类/函数/库：`View`、`Text`、`Image`、`Pressable`、`ScrollView`、`FlatList`、`StyleSheet`。

### 10.3 Expo Router：文件式导航与 Stack/Tabs

Expo Router v4+ 提供基于文件的路由，目录结构即路由结构。`_layout.tsx` 配置 Stack 或 Tabs，`(tabs)/` 分组定义底部标签栏，`[id].tsx` 为动态路由。对应类/函数/库：`expo-router` 的 `<Stack>`、`<Tabs>`、`useRouter`、`useLocalSearchParams`、`Link`。

### 10.4 样式：NativeWind v4（Tailwind for RN）

NativeWind v4 将 Tailwind 的类名语法带入 React Native，通过 Babel 插件与 `global.css` 配置启用。本节覆盖 `className` 在 RN 组件上的使用限制、与 `StyleSheet` 的混合策略，以及响应式前缀在移动端的实际含义。对应类/函数/库：`nativewind`、`tailwindcss`、`babel-plugin-nativewind`。

### 10.5 移动端状态与数据层：Zustand + TanStack Query 的复用

客户端状态继续用 Zustand，服务端状态继续用 TanStack Query，两者的心智模型与 Web 端完全一致。安全存储使用 `expo-secure-store` 替代 `localStorage`。对应类/函数/库：`zustand`、`@tanstack/react-query`、`expo-secure-store`、`expo-image`。

### 10.6 移动端测试：RNTL 与第7章方法的复用

React Native Testing Library 与 React Testing Library 共享"以用户行为为断言中心"的哲学，但查询 API 针对原生组件做了适配。本节演示用 Jest + RNTL 测试移动端页面，并说明与第7章 Web 测试的差异边界。对应类/函数/库：`@testing-library/react-native`、`jest`、`jest-expo`、`fireEvent`、`waitFor`。


## 第11章 项目实战三：移动端应用

> 本项目把第10章的移动端知识收束为一个可上架的应用。选题为**习惯打卡应用（HabitLoop）** ：Expo Router 文件式导航、NativeWind 样式、Zustand 本地状态、TanStack Query 同步云端、本地通知与安全存储。

### 11.1 需求分析与 Expo 项目初始化

明确功能边界（习惯创建、每日打卡、连续天数统计、本地提醒、云端同步），从 `create-expo-app` 出发搭建项目，配置 `app/` 目录、`app.json` 与 EAS 项目。本节给出 Expo 与裸 React Native 的选型对照。对应类/函数/库：`create-expo-app`、`app.json`、`eas-cli`。

### 11.2 文件式路由与 Tabs 导航

用 `(tabs)/_layout.tsx` 定义"今日/统计/设置"三个标签页，用 `habit/[id].tsx` 做习惯详情动态路由，用 `<Stack>` 包裹模态页。本节演示 Expo Router 的布局嵌套与 `Link` 跳转。对应类/函数/库：`expo-router` 的 `<Tabs>`、`<Stack>`、`Link`、`useLocalSearchParams`、`useRouter`。

### 11.3 NativeWind 样式与组件封装

用 NativeWind 的 `className` 构建卡片、按钮、进度环等复用组件，处理 `className` 在 RN 组件上的限制（仅 `View`/`Text`/`Pressable` 等原生组件直接支持），并封装 `cn()` 工具函数合并条件样式。对应类/函数/库：`nativewind`、`className`、`StyleSheet`、`clsx`。

### 11.4 Zustand + 安全存储的本地状态

用 Zustand 管理"今日打卡状态"与"用户偏好"，用 `persist` + `expo-secure-store` 自定义 storage 适配器持久化敏感数据（如同步 token）。对应类/函数/库：`zustand`、`persist`、`createJSONStorage`、`expo-secure-store`。

### 11.5 TanStack Query 云端同步与离线策略

用 `useQuery` 拉取打卡历史、`useMutation` 提交打卡并乐观更新连续天数；配合 `expo-network` 检测离线，用 TanStack Query 的 `onlineManager` 暂停/恢复请求。对应类/函数/库：`useQuery`、`useMutation`、`onlineManager`、`expo-network`。

### 11.6 本地通知、测试与 EAS 构建

用 `expo-notifications` 注册每日提醒；用 RNTL 测试打卡流程与连续天数计算；用 `eas build` 生成 Android/iOS 安装包并分发。本节给出从开发到上架的完整命令清单。对应类/函数/库：`expo-notifications`、`@testing-library/react-native`、`eas build`、`eas submit`。


## 第12章 Remotion：用 React 制作视频

### 12.1 Remotion 的核心模型：帧、Composition 与 useCurrentFrame

Remotion 给开发者一个**帧号**和一块画布。`<Composition>` 声明视频的 `width`、`height`、`fps`、`durationInFrames`；组件内通过 `useCurrentFrame()` 获取当前帧，**所有动画必须由帧号驱动**，CSS 动画和 `setTimeout` 会导致渲染伪影，被明确禁止。对应类/函数/库：`remotion` 的 `Composition`、`useCurrentFrame`、`AbsoluteFill`。

### 12.2 插值与弹簧动画：interpolate 与 spring

`interpolate(frame, inputRange, outputRange, { extrapolateRight: "clamp" })` 是 Remotion 最基础的动画原语；`spring({ frame, fps })` 提供物理感的入场动画。本节用"淡入 + 缩放 + 弹簧滑入"三个场景建立动画手感。对应类/函数/库：`interpolate`、`spring`、`Easing`。

### 12.3 序列、转场与多场景组织

`<Sequence from={} durationInFrames={}>` 组织时间轴上的场景，`<Series>` 顺序播放，`<TransitionSeries>` 配合 `fade()` / `slide()` / `wipe()` 实现场景过渡。本节用"三段式片头"完整演示时间轴编排。对应类/函数/库：`Sequence`、`Series`、`Series.Sequence`、`TransitionSeries`、`fade`、`slide`、`wipe`。

### 12.4 媒体、字幕与渲染输出

`<Audio>` / `<Video>` 从 `@remotion/media` 导入，本地资源用 `staticFile()`；`<Img>` 和 `<AnimatedImage>` 替代原生标签。字幕通过 `createTikTokStyleCaptions()` 生成 token 级高亮。输出使用 `@remotion/renderer` 的 `renderMedia()` 或 AWS Lambda 并行渲染。对应类/函数/库：`Audio`、`Video`、`Img`、`AnimatedImage`、`staticFile`、`createTikTokStyleCaptions`、`renderMedia`、`@remotion/lambda`。


## 第13章 React 游戏开发

### 13.1 2D 游戏：React + PixiJS

PixiJS 提供高性能 2D WebGL/WebGPU 渲染，React 封装层（`@pixi/react`）将精灵、容器、纹理映射为组件。本节用"横向卷轴移动的精灵"演示组件化游戏对象的组织方式，以及 `requestAnimationFrame` 与 React 渲染循环的协同。对应类/函数/库：`pixi.js`、`@pixi/react` 的 `<Stage>`、`<Sprite>`、`<Container>`、`useTick`。

### 13.2 3D 游戏：React Three Fiber

`@react-three/fiber` 将 Three.js 的场景图映射为 React 组件树，`useFrame` 驱动逐帧更新。R3F 不能 SSR，需要 `dynamic(import('R3F'))`；Three.js 会引入较大 chunk，需要路由级代码分割。本节覆盖 `<Canvas>`、`useFrame`、`useThree` 以及基础光照与相机控制。对应类/函数/库：`@react-three/fiber` 的 `Canvas`、`useFrame`、`useThree`、`@react-three/drei` 的 `OrbitControls`、`useTexture`。

### 13.3 游戏状态管理与 60FPS 优化

简单游戏用 `useState`/`useReducer` 足够；复杂游戏引入 Zustand 管理玩家、敌人、分数等全局游戏状态。60FPS 的关键是**最小化 React 重渲染**：游戏循环状态放在 `ref` 中，每帧只更新必要的 UI 切片，重计算用 `useMemo`，后台计算移入 Web Worker。对应类/函数/库：`zustand`、`useRef`、`useMemo`、`useCallback`、Web Worker、`Howler.js`（音频）。

### 13.4 输入处理与移动端适配

键盘用 `useKey` 或原生 `keydown`/`keyup`，触摸用 Pixi/R3F 的事件系统或 `touch` 事件。`devicePixelRatio` 需要显式处理 Retina 屏，`useEffect` 清理必须严格避免内存泄漏。对应类/函数/库：`useKey`（`rooks`）、`window.devicePixelRatio`、`ResizeObserver`。


## 第14章 项目实战四：游戏项目

> 本项目把第13章的 2D/3D 渲染知识收束为一个可玩的完整游戏。选题为**太空射击游戏（StarDrift）** ：PixiJS 渲染 2D 场景、Zustand 管理游戏状态、Web Audio 处理音效、支持键盘与触摸双端操作。

### 14.1 游戏设计与技术选型

明确玩法（玩家飞船射击陨石、分数与生命值、难度递增、游戏结束重开），确定技术栈：`@pixi/react` 渲染、`zustand` 管状态、`Howler.js` 管音效、Vite 构建。本节给出"为什么不用 R3F"的论证（2D 射击用 PixiJS 更轻）。对应类/函数/库：`@pixi/react`、`pixi.js`、`zustand`、`howler`。

### 14.2 Stage、Sprite 与场景搭建

用 `<Stage>` 挂载画布、`<Sprite>` 渲染飞船与陨石、`<Container>` 组织图层；用 `Assets.load` 预加载纹理，用 `<Application>` 配置分辨率与 `devicePixelRatio`。本节演示静态场景到可移动精灵的搭建过程。对应类/函数/库：`Stage`、`Sprite`、`Container`、`Assets`、`Application`。

### 14.3 useTick 游戏循环与状态更新

用 `useTick` 驱动逐帧更新：移动飞船、生成陨石、计算碰撞。关键原则是**帧内状态放 `ref`，跨帧 UI 状态放 Zustand**，避免每帧触发 React 重渲染。本节给出"帧状态 vs 渲染状态"的分离代码结构。对应类/函数/库：`useTick`、`useRef`、`zustand` 的 `getState`/`setState`。

### 14.4 碰撞检测、得分与生命值系统

用 AABB 包围盒做碰撞检测，命中后减少生命值、播放爆炸动画；分数用 Zustand 管理并在 HUD 组件中订阅切片。本节演示"游戏逻辑与渲染分离"的测试友好结构。对应类/函数/库：自定义 `checkCollision`、`zustand` selector、`useStore`。

### 14.5 音效、键盘与触摸双端输入

用 `Howler.js` 播放射击/爆炸音效，用 `useKey` 处理键盘，用 Pixi 的 `pointerdown` 处理触摸，并在移动端渲染虚拟摇杆。本节覆盖 `preload`、`volume` 与音频解锁策略（用户首次交互后播放）。对应类/函数/库：`howler`、`useKey`（`rooks`）、`pointerdown` 事件、自定义 `VirtualJoystick` 组件。

### 14.6 性能优化、测试与发布

用 Pixi 的 `ticker` 控制帧率、用 `Assets` 缓存纹理、用 `useMemo` 缓存碰撞计算；用 Vitest 测试碰撞与计分逻辑（纯函数易测）；最终 `vite build` 部署，并给出移动端性能基线（60FPS、内存占用）。对应类/函数/库：`ticker`、`Assets`、`vitest`、`vite build`、`web-vitals`。


## 第15章 插件、工程化与发布

### 15.1 React 插件化架构：组合模式与插槽

React 的"插件"通常表现为**组合模式**：通过 `children`、render props 或 Context 注入扩展点。本节用"可扩展的 Dashboard 卡片系统"展示如何设计插件接口，使第三方代码可以在不修改核心组件的前提下注入 UI 与行为。对应类/函数/库：`children` 组合、render props、`createContext` + 自定义 `usePlugin` Hook。

### 15.2 构建工具链：Vite 配置、TypeScript 与 ESLint

本节承接 1.1 的 Vite 初始化，深入 `vite.config.ts` 的 React 插件、别名、代理与环境变量，TypeScript 的 `strict` 与 `verbatimModuleSyntax`，以及 ESLint flat config 中的 React Hooks 规则。测试工具链见第7章，不在此重复。对应类/函数/库：`vite`、`@vitejs/plugin-react`、`typescript`、`eslint`、`eslint-plugin-react-hooks`、`eslint-plugin-react-refresh`。

### 15.3 发布与性能基线

Web 端关注路由级代码分割与 bundle 分析；移动端用 EAS Build 构建与分发。本节给出上线前的性能检查清单：LCP、INP、包体积阈值、R3F/Three.js 的懒加载策略，并汇总四个实战项目的部署差异。对应类/函数/库：`React.lazy`、`Suspense`、`rollup-plugin-visualizer`、`expo eas build`、`web-vitals`。

### 15.4 持续集成与持续部署

CI 中串联第7章的测试套件、15.2 的 lint 与类型检查、15.3 的构建与体积分析；部署 Web 端到静态托管或边缘平台，移动端走 EAS Build 分发。本节给出可复用的 GitHub Actions 工作流骨架，并说明四个实战项目如何复用同一套 CI 配置。对应类/函数/库：GitHub Actions、`actions/setup-node`、`pnpm`、`eas-cli`、`lighthouse-ci`。


### 附：章节依赖关系与阅读路径

**技术主干**：第1–3章建立组件、状态与 Hook 心智；第4–6章构成 Web 应用主干（路由、网络、样式）；第7章确立测试方法论。

**项目实战穿插**：
- **第8章 SPA** 收束第1–7章，是第一个"完整应用"里程碑；
- **第9章 多页面应用** 深化第4章路由与第6章样式库，建立复杂架构能力；
- **第10章 移动端技术 → 第11章 移动端项目** 形成"技术+实战"配对；
- **第12章 Remotion** 为独立兴趣模块；
- **第13章 游戏技术 → 第14章 游戏项目** 形成第二组"技术+实战"配对；
- **第15章** 收束工程化与发布，四个项目的 CI 配置在此统一。

**推荐线性路径**：1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10 → 11 → 12 → 13 → 14 → 15。若读者以移动端为目标，可读完第9章后直接进入第10–11章；第12–14章作为选读。四个项目均可独立运行，也可作为课程设计/毕业项目的参考实现。