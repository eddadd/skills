# glodrapay-front-admin-code-cr

代码审查（Code Review）Skill。按 6 步流程对 PR / Commit / 当前工作区更改进行审查，内置规则库、自动生成 `.md` + `.html` 双格式报告，支持历史违规追踪和 OpenSpec 集成。

## 特性

- **双轨规则库**：62 条违规规则项（`CR-RULE-XXX`）+ 9 条整改优化项（`CR-OPT-XXX`），每条规则含等级、描述、检查方式和正反示例
- **6 步审查流程**：前置检查（lint/typecheck）→ 上下文感知 → 规则触发 → 历史检查 → 报告生成 → 结论闭环
- **双格式报告**：Markdown + HTML，可直接归档或嵌入 PR 评论
- **历史追踪**：按提交者统计规则触发频次，同规则触发 3 次以上自动警告
- **重审支持**：增量重审，编号后缀 `-R{N}`，只复查上次问题加新问题
- **3 种触发类型**：`pr`（分支对比）、`commit`（指定提交）、`current`（工作区未提交）
- **OpenSpec 集成**：读取 OpenSpec 变更的 proposal / design / specs，实现范围感知审查

## 安装

```bash
# 将本仓库所有文件拷贝到 AI 助手的 skill 目录
# 例如放在项目根下的某个约定路径中，结构如下：
#
# <project-root>/
# ├── .opencode/skills/code-cr/   ← opencode 路径示例
# └── .skills/code-cr/             ← 其他通用路径也可

# 必需文件：
#   SKILL.md
#   coding-standards.md
#   cr-process.md
#   template/template.md
#   template/template.html
```

## 使用方式

### 前置要求

- 一个支持 Skill 系统或自定义指令的 AI 编程助手
- 项目配置了 `lint`、`typecheck`、`lint:stylelint` 等质量检查命令

### 首次运行

Skill 首次执行时会在项目根自动创建 `code-cr/` 输出目录，并初始化历史记录文件。

### 发起审查

通过 AI 助手发起审查：

#### 直接调该skill发起审查

```
/code-cr
```
或
#### 审查 Pull Request

```
对分支 feature/xxx -> master 做 PR CR
```

#### 审查指定提交

```
对 commits abc1234..def5678 做 commit CR
```

#### 审查当前未提交更改

```
对当前更改做 CR
```

### 输出结构

```
code-cr/
├── index.md                   ← 报告索引
├── .trigger-history.json      ← 规则触发记录
└── YYYYMMDD/
    ├── CR-YYYYMMDD-NNN.md     ← Markdown 报告
    └── CR-YYYYMMDD-NNN.html   ← HTML 报告
```

## 规则库概览

### 违规规则项（62 条，3 个严重等级）

| 分类 | 规则范围 | 示例 |
|------|----------|------|
| 基础编码 | CR-RULE-001 ~ 020 | 箭头函数、禁止 `any`、async 需 try/catch、魔法值抽常量 |
| API 与数据 | CR-RULE-101 ~ 108 | 统一 request 封装、类型定义在 model.d.ts、分页参数规范 |
| 国际化 | CR-RULE-201 ~ 204 | 用户可见文案走 i18n、translateOptions 模式 |
| 样式 | CR-RULE-301 ~ 306 | 优先使用原子类、scoped 样式、引用 Less 变量 |
| 组件与模板 | CR-RULE-401 ~ 416 | 统一表格组件、props 控制弹窗、权限指令、emit 规范 |
| 架构 | CR-RULE-501 ~ 505 | 路由封装、Store 隔离依赖、复用现有组件 |
| 流程 | CR-RULE-901 ~ 902 | 不超范围变更、清理未使用导入 |

### 整改优化项（9 条，建议整改）

| 分类 | 优化项 | 关注点 |
|------|--------|--------|
| 页面与交互 | CR-OPT-001 ~ 005 | 异步请求 loading、表单重置、生命周期调度、权限收敛、组件复用 |
| 可维护性 | CR-OPT-101 ~ 104 | 模板可读性、样式命名、i18n 结构、方法职责拆分 |

## 自定义

### 修改规则库

编辑 `coding-standards.md`：

- 增删规则以匹配你的技术栈
- 调整严重等级（S / M / R）
- 更新违规示例和检查方式
- 添加项目专属优化项

### 修改报告模板

编辑 `template/` 下文件：

- `template.md` — Markdown 报告布局
- `template.html` — HTML 报告布局（含 CSS 样式）

### 配置质量检查命令

修改 `SKILL.md` 中的「环境」章节，替换为项目实际使用的 lint / typecheck 命令。

## 许可

MIT
