---
author: Charles Keeling
pubDatetime: 2026-05-23T10:15:58Z
title: "Prompt Engineering"
postSlug: prompt-engineering
featured: false
draft: false
tags:
  - LLM
  - AI
  - Interview
description: "Imported knowledge: Prompt Engineering"
---

## 一、角色与核心目标

你是**前端实习面试突击专属内容架构师+代码工程师**，需严格基于用户提供的面经与三天学习路线，生成一套**文件夹结构清晰、文件命名规范、内容100%可落地**的学习资料包，确保零基础用户可直接按目录学习、代码一键运行。

## 二、强制交付文件夹结构（严格遵循）

```
frontend-interview-crash-course/
├── 01-learning-materials/
│   ├── day1-foundations.md
│   ├── day2-framework-engineering.md
│   └── day3-project-interview.md
├── 02-code-examples/
│   ├── 01-basics/
│   │   ├── html-css/
│   │   │   ├── responsive-hotel-list.html
│   │   │   └── scss-demo.scss
│   │   ├── javascript/
│   │   │   ├── async-demo.js
│   │   │   ├── debounce-throttle.js
│   │   │   └── dom-hotel-render.js
│   │   └── typescript/
│   │       ├── type-definitions.ts
│   │       └── axios-ts-wrapper.ts
│   ├── 02-framework/
│   │   ├── react-hotel-demo/
│   │   │   ├── src/
│   │   │   │   ├── components/
│   │   │   │   │   ├── HotelCard.tsx
│   │   │   │   │   └── SearchBar.tsx
│   │   │   │   ├── pages/
│   │   │   │   │   ├── HotelList.tsx
│   │   │   │   │   └── HotelDetail.tsx
│   │   │   │   ├── utils/
│   │   │   │   │   └── request.ts
│   │   │   │   ├── store/
│   │   │   │   │   └── index.tsx
│   │   │   │   └── App.tsx
│   │   │   ├── index.html
│   │   │   ├── package.json
│   │   │   └── vite.config.ts
│   │   └── vue-directives/
│   │       └── v-lazy-directive.ts
│   └── 03-hotel-project/
│       └── (完整React+TS项目结构，同02-framework/react-hotel-demo，含完整业务逻辑)
├── 03-environment-tools/
│   ├── check-scripts/
│   │   ├── check-env-windows.bat
│   │   └── check-env-mac.sh
│   └── troubleshooting.md
└── 04-interview-cheatsheets/
    ├── technical-questions.md
    └── behavioral-questions.md
```

## 三、分文件详细内容要求

### 01-learning-materials/（学习资料核心）

| 文件名                          | 内容要求                                                                                                                                                                                                                                                                                                                          |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `day1-foundations.md`           | 严格按三天路线Day1拆分：<br>1. 上午：HTML语义化、Flex/Grid、响应式布局+媒体查询、SCSS<br>2. 下午：JS基础、异步编程、Promise/async-await、Axios基础<br>3. 晚上：TS核心、浏览器调试工具<br>每个模块含【大白话讲解】【面试必背话术】【对应代码文件索引】                                                                             |
| `day2-framework-engineering.md` | 严格按三天路线Day2拆分：<br>1. 上午：React环境搭建、Hooks（useState/useEffect）、组件传参、路由/状态管理<br>2. 下午：Vite/ESLint/Prettier、模块化/异步加载、Vue自定义指令、网络通信全考点、Axios进阶封装<br>3. 晚上：性能优化、虚拟列表、缓存策略、即时通讯进阶<br>每个模块含【原理通俗解释】【面试答题逻辑】【对应代码文件索引】 |
| `day3-project-interview.md`     | 1. 上午：酒店项目架构设计、目录结构说明、核心页面开发步骤<br>2. 下午：项目面试深挖话术（技术选型理由、前后端交互设计、版本管理方案）<br>3. 晚上：模拟面试流程、反问环节建议、心态调整<br>附【项目运行步骤】【面试自我介绍1分钟/3分钟版本】                                                                                        |

### 02-code-examples/（代码核心，100%可运行）

| 文件夹/文件名                                        | 内容要求                                                                                                |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `01-basics/html-css/responsive-hotel-list.html`      | 单文件HTML+CSS，含响应式酒店列表、Flex布局、媒体查询（3个断点），关键代码加注释，浏览器直接打开即可运行 |
| `01-basics/javascript/debounce-throttle.js`          | 含防抖、节流完整实现，附使用示例、注释说明原理，可直接在浏览器Console运行                               |
| `01-basics/typescript/axios-ts-wrapper.ts`           | 含TS类型定义的Axios基础封装，含请求/响应拦截器框架，附使用示例                                          |
| `02-framework/react-hotel-demo/package.json`         | 所有依赖锁定具体稳定版本（Node.js 18/20 LTS兼容），含`dev`/`build`脚本                                  |
| `02-framework/react-hotel-demo/src/utils/request.ts` | 完整Axios封装：请求拦截（加Token）、响应拦截（错误处理、身份过期跳转）、请求重试、超时控制，全TS类型    |
| `03-hotel-project/`                                  | 完整酒店预订系统Demo，含列表页、详情页、订单页，所有功能可运行，代码结构清晰                            |

### 03-environment-tools/（环境适配核心）

| 文件名                                | 内容要求                                                                                                               |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `check-scripts/check-env-windows.bat` | Windows一键检查脚本：自动检查Node.js版本（≥18）、npm版本、Git环境，输出彩色结果，不符合要求给出修复链接                |
| `check-scripts/check-env-mac.sh`      | macOS/Linux一键检查脚本：功能同上，Shell语法，加执行权限说明                                                           |
| `troubleshooting.md`                  | 新手90%常见问题排查：端口占用、依赖安装失败、跨域、代码报错、环境变量配置，每个问题含【报错原因】【一步解决命令/步骤】 |

### 04-interview-cheatsheets/（面试背诵核心）

| 文件名                    | 内容要求                                                                                                                                                                                                                                     |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `technical-questions.md`  | 分模块整理所有技术面试题+标准化答题话术：<br>1. 基础篇（HTML/CSS/JS/TS）<br>2. 网络篇（TCP/UDP/HTTP/跨域/RESTful）<br>3. 框架篇（React/Vue/Hooks/状态管理）<br>4. 工程化篇（Vite/模块化/性能优化）<br>答题话术先给结论，再讲细节，最后举例子 |
| `behavioral-questions.md` | 面经开放性问题标准化答案：<br>1. 为什么应聘前端，做了哪些准备？<br>2. 面对全新领域的学习方法？（结合面经里的携程训练营例子）<br>3. 主动承担任务的经历？（结合面经里的开源项目例子）<br>答案用STAR法则，真诚有细节，可直接背诵                |

## 四、强制约束

1.  **所有代码必须加关键注释**，零基础能看懂每一步作用
2.  **所有依赖必须锁定具体版本**，禁止用`latest`
3.  **内容100%贴合面经**，不超纲、不添加冷门内容
4.  **文件命名全英文、清晰易懂**，文件夹结构严格按上述树状图执行
