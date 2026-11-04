# Agent Skills

[![skills.sh](https://skills.sh/b/eddadd/skills)](https://skills.sh/eddadd/skills)

一组面向 AI 编码 Agent 的 Skill 集合，遵循 [Agent Skills 标准](https://agentskills.io/specification)：每个目录包含一个 `SKILL.md`（YAML frontmatter + Markdown 指令），可选的 `references/`、`scripts/`、`templates/` 提供按需加载的补充材料。

## 安装

```bash
# 查看仓库内所有 skill
npx skills add eddadd/skills --list

# 安装全部
npx skills add eddadd/skills --all

# 只安装指定的几个
npx skills add eddadd/skills --skill glodrapay-front-test --skill glodrapay-front-admin-git-commit

# 全局安装（对所有项目可用）
npx skills add eddadd/skills -g
```

安装完成后可用 `npx skills list` 查看已安装项，`npx skills update` 拉取本仓库最新版本。

## Skill 清单

### 通用

| Skill | 说明 |
| --- | --- |
| [glodrapay-front-test](./glodrapay-front-test) | 前端项目分级测试。默认标准模式按指定范围或变更范围测试，明确要求全量测试时才执行完整验证链；两种模式均生成测试报告。适配 Vue / React + Vitest / Jest / Playwright / Cypress。 |
| [glodrapay-front-admin-code-cr](./glodrapay-front-admin-code-cr) | 代码审查。按 6 步流程审查 PR / Commit / 当前更改，引用内置 `coding-standards.md` 规则，输出 `.md` + `.html` 报告到项目根 `code-cr/`。 |
| [glodrapay-front-admin-git-commit](./glodrapay-front-admin-git-commit) | 基于 Conventional Commits 生成三段式提交信息（Header + Body + Footer），自动分析 diff 推断 type 与 scope，支持中文。 |
| [openspec-spec-methodology](./openspec-spec-methodology) | OpenSpec spec 增强方法论。检测 `opsx-propose` / `opsx-new` / `opsx-ff` 等命令时自动触发，对需求做复杂拆解、颗粒度校准、边界铺排，结论作为上下文注入，产出物仍由 OpenSpec 生成。 |

### GlodraPay 内部流程

这几个 skill 绑定 GlodraPay 内部约定（文档站点、云效流水线、openspec 工作流），外部项目通常无法直接使用。

| Skill | 说明 |
| --- | --- |
| [glodrapay-ai-testing-qc](./glodrapay-ai-testing-qc) | AI 测试质检。消费 openspec 产出物（`tasks.md` 验收标准 + `spec.md` 场景）转成测试断言，双层验证（浏览器实测 + 代码质量走查），输出分级测试报告。 |
| [glodrapay-web-docs-update](./glodrapay-web-docs-update) | VitePress 文档站点维护。新增 API / 普通文档 / 地区文档，或按描述更新已有内容。 |
| [glodrapay-pipeline-run](./glodrapay-pipeline-run) | 云效流水线运行助手。按 test / beta / prod 分组逐级点选流水线与分支后运行。 |

## 仓库结构

```
skills/
├── <skill-name>/
│   ├── SKILL.md          # 必需：frontmatter(name/description) + 指令正文
│   └── references/       # 可选：按需加载的详细文档
```

`SKILL.md` 的 `name` 必须与目录名一致，仅允许小写字母、数字和连字符。

## 安全提示

安装第三方 skill 前请先阅读其内容，尤其是 `scripts/` 目录下的脚本——它们会执行命令。本仓库的 skill 均为纯 Markdown 指令，不含可执行脚本。

## License

[MIT](./LICENSE)
