# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **重要提示**: 本文件是任何 AI Agent（Claude、Gemini、Cursor 等）在此仓库工作的唯一真实来源，汇总了架构上下文、开发工作流和行为准则。

---

## 1. 核心理念与规范

### 核心哲学

- **渐进式推进优于大爆炸式改动**: 将复杂任务拆分为可管理的阶段
- **学习现有代码**: 实现新功能前先理解现有模式
- **清晰意图优于聪明代码**: 优先考虑可读性和可维护性
- **简单优于复杂**: 保持实现简洁直接 - 优先解决问题和维护便利性

### 八荣八耻

- 以**认真研究**为荣，以**猜测 API**为耻
- 以**寻求确认**为荣，以**模糊执行**为耻
- 以**人工验证**为荣，以**假设业务逻辑**为耻
- 以**复用现有接口**为荣，以**创造新接口**为耻
- 以**主动验证**为荣，以**跳过测试**为耻
- 以**遵守规范**为荣，以**破坏架构**为耻
- 以**诚实承认无知**为荣，以**假装理解**为耻
- 以**谨慎重构**为荣，以**盲目修改**为耻

### 代码质量标准

- **仅使用英文注释**: 严禁使用中文注释
- **避免不必要的注释**: 简单明显的代码让代码自解释
- **自文档化代码**: 优先使用显式类型和清晰命名，而非行内文档
- **组合优于继承**: 在适用场景下优先使用函数式模式（Rust）

---

## 2. 项目概述

**名称**: Pake
**用途**: 一键将任意网页转换为轻量级桌面应用（支持 macOS、Windows、Linux）
**核心价值**: 比 Electron 打包小约 20 倍，通常仅 5MB 左右
**实现机制**: 使用 Rust 和 Tauri 将网页内容封装在系统 WebView 中（Windows 用 WebView2，macOS/Linux 用 WebKit）

---

## 3. 技术栈

- **核心框架**: [Tauri v2](https://tauri.app/) (Rust)
- **CLI 工具**: TypeScript, Node.js, Commander.js
- **前端**: HTML/CSS/JS（注入到 WebView 中）
- **构建工具**: Rollup（CLI 构建用）
- **包管理器**: pnpm
- **最低版本要求**: Rust >= 1.85, Node >= 22

---

## 4. 仓库架构

### 目录结构

```
bin/                    # CLI 工具源代码
├── cli.ts              # 入口文件
├── defaults.ts         # CLI 参数默认值
├── types.ts            # TypeScript 类型定义
├── builders/           # 应用构建逻辑
│   ├── BaseBuilder.ts  # 构建器基类
│   ├── BuilderProvider.ts
│   ├── MacBuilder.ts   # macOS 平台构建
│   ├── WinBuilder.ts   # Windows 平台构建
│   └── LinuxBuilder.ts # Linux 平台构建
├── options/            # CLI 参数处理
│   ├── index.ts        # 参数解析入口
│   └── icon.ts         # 图标处理
├── helpers/            # 辅助函数
└── utils/              # 工具函数

src-tauri/              # Tauri 桌面应用
├── src/
│   ├── main.rs         # Rust 入口，调用 lib.rs
│   ├── lib.rs          # **核心逻辑**：菜单设置、插件初始化、应用生命周期
│   ├── util.rs         # 工具函数
│   ├── app/            # 应用模块（重要！）
│   │   ├── mod.rs      # 模块声明
│   │   ├── config.rs   # 应用配置结构体定义
│   │   ├── invoke.rs   # Tauri invoke 命令处理
│   │   ├── menu.rs     # 应用菜单设置
│   │   ├── setup.rs    # 插件初始化配置
│   │   └── window.rs   # 窗口创建和管理
│   └── inject/         # 注入到目标网页的 JavaScript
│       ├── event.js    # 事件处理（快捷键、导航等）
│       ├── component.js # UI 组件注入
│       ├── style.js    # 样式覆盖
│       ├── auth.js     # 认证相关
│       └── custom.js   # 自定义脚本（用户可编辑）
├── tauri.conf.json     # 默认配置
├── pake.json           # Capabilities 配置
└── Cargo.toml          # Rust 依赖配置

dist/                   # 编译后的 CLI 输出
```

---

## 5. 核心工作流

### 开发流程

1. **理解**: 研究代码库中的现有模式
2. **规划**: 将复杂工作分解为阶段
3. **测试**: 适用时先编写测试
4. **实现**: 最小化可工作方案
5. **重构**: 优化和清理

### 常用命令

```bash
# 安装依赖
pnpm i

# CLI 开发（监听模式，重新编译 TypeScript）
pnpm run cli

# 应用开发（热重载）
pnpm run dev

# 构建 CLI（Rollup 编译）
pnpm run cli:build

# 构建应用
pnpm run build

# 调试构建
pnpm run build:debug

# 运行测试（先构建 CLI，使用 PAKE_CREATE_APP=1 环境变量）
pnpm test

# 代码格式化
pnpm run format
```

### 发布流程

- CI/CD 通过 GitHub Actions 处理发布
- 版本号管理于 `package.json` 和 `src-tauri/tauri.conf.json`

---

## 6. 实现细节

### CLI 工具 (`bin/`)

- **参数解析**: 使用 `commander` 库
- **构建器选择**: `BuilderProvider` 根据平台选择正确的构建器
- **继承体系**: `BaseBuilder` → `MacBuilder`/`WinBuilder`/`LinuxBuilder`
- **关键参数**: `--width`, `--height`, `--icon`, `--inject`, `--hide-title-bar`

### Tauri 应用 (`src-tauri/`)

#### 模块职责 (`src/app/`)

| 文件 | 职责 |
|------|------|
| `config.rs` | 定义 `AppConfig`、`WindowConfig` 结构体，存储应用配置 |
| `invoke.rs` | 处理前端调用的 Rust 命令（如下载文件、发送通知） |
| `menu.rs` | 构建应用菜单（区分 macOS 和其他平台） |
| `setup.rs` | 初始化 Tauri 插件（window-state、oauth、http 等） |
| `window.rs` | 创建和管理 WebView 窗口 |

#### `lib.rs` 核心功能

- 设置菜单（macOS 与其他平台不同）
- 配置插件：`window-state`、`single-instance`、`opener`、`notification` 等
- 处理自定义命令：`download_file`、`send_notification`
- 管理窗口事件（关闭时隐藏而非退出）

#### 窗口管理

- 使用 `tauri-plugin-window-state` 记住窗口尺寸和位置
- 支持"启动到托盘"和"关闭时隐藏"

### 注入系统

**路径**: `src-tauri/src/inject/`

**机制**: Tauri 在运行时注入这些脚本到目标网页

**功能**:
- 自定义 CSS 覆盖
- 键盘快捷键（缩放、导航）
- 平台特定调整

---

## 7. 常见 AI 任务

### 添加 CLI 参数

1. 更新 `bin/types.ts`（接口定义 `PakeCliOptions`）
2. 更新 `bin/defaults.ts`（默认值）
3. 更新 `bin/cli.ts`（Commander 选项定义）
4. 更新 `bin/options/index.ts`（参数处理逻辑）
5. 如需传递到 Rust，更新 `src-tauri/src/app/config.rs`（结构体定义）

### 修改应用菜单

编辑 `src-tauri/src/app/menu.rs`

### 修改注入脚本

编辑 `src-tauri/src/inject/event.js` 或 `component.js`

### 添加新的 Tauri 命令

1. 在 `src-tauri/src/app/invoke.rs` 中定义命令处理函数
2. 在 `src-tauri/src/lib.rs` 的 `invoke_handler` 中注册命令
3. 前端通过 `invoke()` 调用

### 修改窗口配置

1. TypeScript 侧: `bin/types.ts` 中的 `WindowConfig` 接口
2. Rust 侧: `src-tauri/src/app/config.rs` 中的结构体
3. 应用配置: `src-tauri/src/app/window.rs` 中的窗口创建逻辑
