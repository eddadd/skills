# CR 审查报告

| 字段 | 内容 |
|------|------|
| **CR 编号** | CR-{{DATE}}-{{SEQ}} |
| **提交者** | @{{SUBMITTER}} |
| **审查者** | @{{REVIEWER}} |
| **审查日期** | {{DATE}} |
| **审查范围** | {{SCOPE}} |
| **分支 / 提交** | {{BRANCH}}{{TARGET_BRANCH}} |
| **PR 链接** | {{PR_URL}} |
| **变更摘要** | {{SUMMARY}} |
| **重审来源** | {{RE_REVIEW_SOURCE}} |

---

## OpenSpec 关联

| 字段 | 内容 |
|------|------|
| 关联变更 | {{CHANGE_NAME}} |
| 变更范围 | {{CHANGE_SCOPE}} |
| 关联规范 | {{SPEC_FILES}} |

### 场景覆盖检查

| 场景 | 状态 |
|------|------|
{{SCENARIO_COVERAGE}}

---

## 一、前置检查结果

| 检查项 | 状态 |
|--------|------|
| `pnpm lint` | {{LINT_STATUS}} |
| `pnpm typecheck` | {{TYPECHECK_STATUS}} |
| `pnpm lint:stylelint` | {{STYLELINT_STATUS}} |

---

## 二、问题清单

### 🔴 违规规则项

| 规则 ID | 等级 | 描述 | 文件 | 行号 |
|---------|------|------|------|------|
{{DIRECT_TRIGGERS}}

### 🟠 整改优化项

| 优化项 ID | 优先级 | 描述 | 文件 | 行号 | 说明 |
|-----------|--------|------|------|------|------|
{{OPTIMIZATION_ITEMS}}

### ❓ 需人工审查项

| 序号 | 描述 | 文件 | 行号 | 不确定原因 |
|------|------|------|------|------------|
{{MANUAL_REVIEW_ITEMS}}

### 🟢 已确认通过项（未触发）

{{PASSED_RULES}}

---

## 三、历史重复触发提醒

{{HISTORICAL_REMINDERS}}

---

## 四、审查结论

- [ ] ✅ **批准通过** — 可合并 / 可提交
- [ ] 🔄 **需整改后重审** — 请在 {{DEADLINE}} 前完成修改
- [ ] ❌ **驳回** — 原因：{{REJECT_REASON}}

### 审查意见

{{REVIEW_COMMENTS}}

---

*报告自动生成于 {{GENERATED_TIME}}*
*模板版本: 1.2*
