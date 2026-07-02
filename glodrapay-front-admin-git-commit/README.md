# glodrapay-front-admin-git-commit

基于 Conventional Commits 规范生成标准化 Git 提交信息。自动分析 diff 推断 type 和 scope，生成三段式提交信息（Header + Body + Footer），支持中文提交信息。

## 特性

- **Conventional Commits 规范**：严格遵循 `<type>(<scope>): <subject>` 格式
- **自动推断 Type**：从 diff 内容智能判断提交类型（feat / fix / docs / style / refactor / perf / test / chore / build / ci / revert）
- **自动推断 Scope**：从变更文件路径推断所属模块，支持中文 scope
- **三段式结构**：Header（必填）+ Body（可选）+ Footer（可选），根据改动规模自动决定是否包含 Body/Footer
- **混合变更检测**：检测暂存区是否混合多种类型变更，建议拆分提交
- **中文提交信息**：与项目现有提交风格一致
- **安全护栏**：不自动执行 git commit，不泄露 secrets，不写废话

## 安装

将本目录所有文件拷贝到 AI 助手的 skill 目录：

```
<project-root>/
└── .agents/skills/glodrapay-front-admin-git-commit/
    ├── SKILL.md     ← skill 定义与执行入口
    ├── meta.json    ← skill 元数据
    └── README.md    ← 说明文档
```

## 使用方式

### 触发方式

- `/glodrapay-front-admin-git-commit`
- "帮我写个 commit"
- "生成提交信息"
- "commit message"

### 输出

生成提交信息文本，供用户确认后自行提交。例如：

```
feat(用户管理): 新增批量导入功能

- 后端接口新增 /users/batch-import 端点
- 前端新增导入按钮及进度提示
- 支持按状态筛选导入
```

### 执行步骤

1. **收集变更**：读取 git status / diff --cached / diff / log
2. **分析变更**：逐文件分析 diff，推断 type 和 scope
3. **推断 scope**：从文件路径映射到业务模块名
4. **生成信息**：按规模选择 Header / Header+Body / Header+Body+Footer
5. **输出结果**：展示生成的提交信息，提供 git commit 命令模板

## Conventional Commits 类型

| 类型       | 说明              | 版本影响  |
| -------- | --------------- | ----- |
| feat     | 新功能             | Minor |
| fix      | Bug 修复          | Patch |
| docs     | 文档变更            | —     |
| style    | 格式调整            | —     |
| refactor | 代码重构            | —     |
| perf     | 性能优化            | Patch |
| test     | 测试变更            | —     |
| chore    | 构建/辅助工具变更       | —     |
| build    | 构建系统/外部依赖变更     | —     |
| ci       | CI 配置变更         | —     |
| revert   | 回退提交            | —     |

## 许可

MIT
