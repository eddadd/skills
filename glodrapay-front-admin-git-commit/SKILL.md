---
name: glodrapay-front-admin-git-commit
description: 基于 Conventional Commits 规范生成标准化 Git 提交信息。自动分析 diff 判断类型与范围，生成三段式提交信息（Header + Body + Footer），支持中文提交信息。
license: MIT
metadata:
  author: syf
  version: "1.0"
---

# git-commit

基于 Conventional Commits 规范生成标准化 Git 提交信息。分析当前 diff 或暂存区变更，自动推断提交类型（type）和范围（scope），生成符合规范的三段式提交信息。

> 本 skill 只生成提交信息文本，不执行 git commit 或 git push 命令。

***

## 前导

### 环境

- 项目根：`.`（Git 仓库根目录）
- 包管理：`pnpm`
- 中文回复，每次回答前固定加称呼：`老板`

### Conventional Commits 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

- Header（必填）：`<type>(<scope>): <subject>`
- Body（可选）：复杂改动时说明"为什么改、怎么改"
- Footer（可选）：标记 BREAKING CHANGE 或关联 Issue

***

## 提交类型 (Type)

| 类型       | 说明                     | 版本影响    |
| -------- | ---------------------- | ------- |
| feat     | 新功能开发                  | Minor   |
| fix      | Bug 修复                 | Patch   |
| docs     | 仅文档变更，不影响代码逻辑          | —       |
| style    | 代码格式调整（不影响运行结果）        | —       |
| refactor | 代码重构（既非新功能也非修 Bug）     | —       |
| perf     | 性能优化                   | Patch   |
| test     | 测试代码变更                 | —       |
| chore    | 构建过程或辅助工具变更            | —       |
| build    | 构建系统或外部依赖变更            | —       |
| ci       | CI 配置文件和脚本变更           | —       |
| revert   | 回退之前提交                 | —       |

***

## 写作规范

### Subject 规则

- 使用祈使句，现在时态："新增" / "修复" / "移除"，不用"新增了"/"修复了"/"移除了"
- 首字母小写（英文时）；中文不受此限制
- 不超过 50 个字符
- 不以句号结尾
- 直接说明"改了什么"，不写"This commit does..."

### Body 规则

- **仅在改动复杂、subject 无法自解释时才写 Body**
- 说明"为什么改"和"怎么改"，不重复 subject 已说过的内容
- 每行不超过 72 字符
- 使用 `-` 列表项，不用 `*`
- 与 Header 之间空一行

### Footer 规则

- **仅在以下情况才写 Footer**：
  - 不兼容变动：`BREAKING CHANGE: <描述>`
  - 关联 Issue：`Closes #123` 或 `Refs #17`
  - 回退提交：`Reverts <commit-hash>`
- 与 Body 之间空一行；无 Body 时与 Subject 空一行

### Scope 推断规则

从变更文件路径推断 scope：

| 路径模式                | scope 示例    |
| ------------------- | ---------- |
| `src/views/xxx/`    | xxx 模块名    |
| `src/api/xxx/`      | xxx 接口     |
| `src/components/`   | 组件         |
| `src/stores/`       | 状态管理       |
| `src/router/`       | 路由         |
| `src/assets/styles/` | 样式         |
| `src/i18n/`         | 国际化        |
| `docs/`             | 文档         |
| 根目录配置文件             | config     |

- 涉及多个模块时取主要变更模块，或使用组合 scope：`(用户+订单)`
- 无法确定时省略 scope，格式变为 `<type>: <subject>`
- scope 使用中文模块名，与项目现有提交风格保持一致

### Type 推断规则

按变更内容推断 type：

1. **新增页面/组件/API/功能** → `feat`
2. **修复已知 Bug** → `fix`
3. **仅修改 .md / 注释** → `docs`
4. **仅格式/空格/分号调整** → `style`
5. **代码结构优化但不改变功能** → `refactor`
6. **性能相关改动** → `perf`
7. **测试文件变更** → `test`
8. **构建配置/依赖/工具** → `chore` / `build` / `ci`
9. **回退之前提交** → `revert`
10. **同时有新功能+Bug 修复** → 优先取 `feat`，除非修复更关键则取 `fix`

***

## 触发条件

本 skill 在以下场景被调用：

- 用户请求生成提交信息："帮我写个 commit"、"生成提交信息"、"commit message"
- 用户暂存了文件并准备提交
- 用户说"提交这些改动"

***

## 执行步骤

***

### Step 1：收集变更信息

```bash
git status
git diff --cached
git diff
git log --oneline -5
```

- `git status` → 确认暂存区和工作区状态
- `git diff --cached` → 读取已暂存变更的详细内容
- `git diff` → 读取未暂存变更的详细内容（如有）
- `git log --oneline -5` → 了解最近提交风格，保持一致性

**如果没有暂存变更**：
- 通知用户：`老板，当前暂存区没有变更。请先用 git add 暂存需要提交的文件，或者告诉我需要暂存哪些文件。`
- 可建议按逻辑分组暂存，避免一次提交混合多种类型变更

***

### Step 2：分析变更内容

逐文件分析 diff 内容：

1. **变更文件列表** → 推断 scope
2. **变更性质** → 推断 type（参照 Type 推断规则）
3. **变更规模**：
   - 小改动（1-3 个文件，逻辑简单）→ 只写 Header
   - 中等改动（4-10 个文件）→ Header + Body
   - 大改动（10+ 文件或跨模块）→ Header + Body + Footer（如有 breaking change 或关联 issue）

4. **检查是否混合类型**：
   - 如果一次暂存了 feat + fix + docs 等混合变更 → 建议拆分为多次提交
   - 建议格式：`老板，这次暂存包含了多种类型变更（feat + fix），建议拆分为多次提交以保证提交历史清晰。是否需要我帮你分组暂存？`

***

### Step 3：推断 scope

从变更文件路径推断 scope：

- 读取所有变更文件路径
- 按上述"Scope 推断规则"映射
- 如果涉及 `src/views/用户管理/` 多个文件 → scope = `用户管理`
- 如果涉及 `src/api/` + `src/views/` 跨层 → scope = 主要业务模块名
- 根目录配置文件变更 → scope = `config`

***

### Step 4：生成提交信息

按以下模板生成：

#### 简单改动（仅 Header）

```
<type>(<scope>): <subject>
```

示例：
```
feat(用户管理): 新增批量导入功能
fix(支付): 修复退款金额计算精度问题
docs: 更新部署流程说明
chore(config): 升级 vite 到 6.0
```

#### 中等改动（Header + Body）

```
<type>(<scope>): <subject>

<body-line-1>
<body-line-2>
```

示例：
```
feat(订单): 新增订单导出 Excel 功能

- 后端接口新增 /orders/export 端点
- 前端新增导出按钮及进度提示
- 支持按日期范围和状态筛选导出
```

#### 大改动（Header + Body + Footer）

```
<type>(<scope>): <subject>

<body>

BREAKING CHANGE: <描述>
Closes #123
```

示例：
```
refactor(权限): 重构路由权限校验逻辑

- 从 routerGuards 中抽取权限判断为独立模块
- 统一 usePerMenu 与本地路由的权限合并策略
- 移除废弃的白名单路由配置

BREAKING CHANGE: 路由权限配置格式从数组改为对象，需更新所有页面路由声明
Closes #42
```

***

### Step 5：输出提交信息

将生成的提交信息直接输出给用户，格式如下：

```markdown
老板，根据当前变更生成的提交信息：

```
<完整提交信息文本>
```

变更概况：X 个文件，主要涉及 <scope>。
```

**如果用户需要直接提交**，提供命令模板：

```bash
git commit -m "<type>(<scope>): <subject>" -m "<body>" -m "<footer>"
```

> 注意：不主动执行 git commit，只生成信息供用户确认后自行提交。除非用户明确说"帮我提交"，才执行 git commit。

***

## 护栏

- 只生成提交信息，不自动执行 git commit（除非用户明确要求）
- 不猜测用户意图，diff 不清晰时提问
- 不在提交信息中泄露 secrets、密码、token
- 混合类型变更时建议拆分提交
- 提交信息使用中文（与项目现有风格一致）
- scope 优先使用中文模块名
- Subject 不超过 50 字符
- Body 每行不超过 72 字符
- 不写废话："This commit does..."、"修改了一些文件..."、"update code..."
- 不写 AI 归属标记
- 不重复文件名（diff 已包含）

***

## 混淆协议

当以下情况发生时，通过 AskUserQuestion 向用户澄清：

1. **暂存区为空**：无变更可分析，询问用户是否需要暂存
2. **混合类型变更**：暂存区包含多种类型改动，询问是否拆分
3. **scope 不确定**：变更跨多个模块，无法判断主要 scope
4. **type 不确定**：变更性质模糊（重构 vs 新功能 vs 修复）

**禁止**：
- 猜测 type/scope
- 将明显的新功能标为 refactor
- 将 Bug 修复标为 feat
- 在不确定时强行生成提交信息而不提问
