---
name: glodrapay-web-docs-update
description: GlodraPay VitePress 文档站点维护助手。用于对 glodrapay-web-docs 项目进行新增和更新操作。新增分为三类：新增 API 接口文档、新增普通文档、新增地区文档；更新则根据用户提供的描述信息修改已有内容。每次运行该 skill 时必须先询问用户本次变更是新增（API/文档/地区三选一）还是更新原有内容，然后根据选择走对应流程。当用户说"维护文档"、"更新文档"、"新增API"、"新增文档"、"新增地区"、"添加接口文档"、"添加地区文档"、"glodrapay docs 变更"等场景时使用本 skill。强烈建议在涉及 glodrapay-web-docs 项目的任何文档增删改任务中都优先触发本 skill。
license: MIT
metadata:
  author: syf
  version: "1.0"
---

# glodrapay-web-docs-update

GlodraPay VitePress 文档站点维护助手。用于对项目进行新增和更新操作。

## 触发场景

- 用户要求新增/更新 GlodraPay 文档站点内容
- 用户提到"新增API"、"新增文档"、"新增地区"、"更新文档"、"维护文档"
- 在 glodrapay-web-docs 仓库中执行任何文档增删改任务

## 执行流程

每次运行本 skill 都必须严格按以下四步顺序执行，不可跳步：

### 第一步：拉取最新代码

先拉取当前分支最新代码，避免在过期代码上工作造成冲突：

```bash
git fetch origin
git status
git pull --ff-only
```

- 如有未提交的本地改动，提示用户先处理（commit/stash），不要自动 commit
- 如 pull 失败（冲突/无上游分支），暂停并向用户报告，不强行处理

### 第二步：询问本次变更类型

**使用弹窗（`question` 工具）询问**，不要用纯文本提问后暂停等待。向用户弹窗展示四个选项：

1. **新增 API** — 新增接口文档
2. **新增文档** — 新增普通指南/开发文档
3. **新增地区** — 新增地区参数文档
4. **更新原有内容** — 根据描述信息修改已有内容

根据用户的选择执行对应流程：
- 选择 1/2/3：读取对应的专属 prompt，按其要求**用弹窗逐项**向用户索要必填信息（每项都用 `question` 工具弹窗，不暂停会话等待文本输入）
- 选择 4：根据用户本次输入的描述信息直接进行更新

### 第三步：执行代码修改

根据用户选择，读取对应的专属 prompt 文件并严格按其步骤执行：

| 变更类型 | 专属 prompt 文件 | 必填信息 |
|---------|----------------|---------|
| 新增 API | `references/add-api.md` | OpenAPI JSON、文件夹名称 |
| 新增文档 | `references/add-doc.md` | 中文标题、文档内容、文件夹名称（可选） |
| 新增地区 | `references/add-region.md` | 地区名（中文）、参数文件列表、基础文档配置 |

> **新增地区提示**：用户可能不知道"基础文档配置"（`docsList` 和 `modules`）怎么填。在向用户索要此项信息前，先在弹窗的描述中提示用户参考语雀文档：https://fshows.yuque.com/tech-ozd0u/suyx8h/cxn4q2tlrs9w5e51#UWRE0 ，再根据用户的输入和项目已有配置（参考 `docs/.vitepress/theme/constants/common.ts` 中的 `LOCALES`）协助用户确定 `docsList` 和 `modules` 的内容。

### 第四步：质量检查

代码修改完成后，必须执行以下检查：

1. **格式检查**：MD 文件 frontmatter、script setup 语法、Vue 组件用法符合项目规范
2. **构建验证**：运行 `npm run build` 确认生产构建成功（构建过程含类型检查，比单独跑 vue-tsc 更可靠，且无需联网安装 vue-tsc）
3. **需求完成度**：对照专属 prompt 的输出要求逐项核对

### 第五步：暂存并生成提交信息

将修改加入暂存区：

```bash
git add -A
```

然后调用 `glodrapay-front-admin-git-commit` skill 生成标准化的 Conventional Commits 提交信息：

- 如果当前环境有 `glodrapay-front-admin-git-commit` skill 可用，**优先调用它** 生成提交信息
- 如果该 skill 不可用，则自己按 Conventional Commits 规范生成提交信息（中文，含 type/scope/subject）

生成提交信息后：
- **不主动执行 git commit**，除非用户明确要求
- 将提交信息和变更概况呈现给用户确认

## 重要约定

### 文件结构

```
docs/
├── zh/                    # 中文（默认语言，经 rewrites 映射到根路径）
│   ├── api/               # API 文档（按模块分文件夹）
│   ├── guide/             # 指南文档
│   ├── region/            # 地区文档（按地区分文件夹）
│   └── themeConfig.ts     # 中文侧边栏/导航配置
├── en/                    # 英文（结构同 zh/）
├── config.ts              # 中文 locale 配置
├── en/config.ts           # 英文 locale 配置
└── .vitepress/
    └── theme/
        └── constants/
            └── common.ts # LOCALES 国际化常量
```

### i18n 双语同步原则

所有新增/更新的用户可见文本必须同时提供中英文两个版本：
- `common.ts` 中 `LOCALES[LANGUAGE.zh]` 和 `LOCALES[LANGUAGE.en]` 同步添加
- `zh/themeConfig.ts` 和 `en/themeConfig.ts` 同步更新
- 文件内容中文版用中文，英文版用英文

### 文件名规范

- 一律使用 **小驼峰格式**（camelCase）
- 键名（common.ts 中的键）与文件名保持一致（不含 `.md`）
- 新增前检查同名文件/键是否已存在，避免重复

### 中文沟通约定

本 skill 在与用户对话时使用中文，保持简洁专业的风格。

## 护栏

- 不主动 git commit / git push，除非用户明确要求
- 不猜测用户意图，必填信息不清楚时一个一个问清楚
- 不跳过拉取代码、质量检查环节
- 每次新增/更新都要保证中英文双语同步
- 修改配置文件（common.ts/themeConfig.ts/config.ts）前先读取当前内容，避免覆盖
