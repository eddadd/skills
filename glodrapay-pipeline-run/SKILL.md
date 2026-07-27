---
name: glodrapay-pipeline-run
description: GlodraPay 流水线运行助手。用户只需要提供分组名（test/beta/prod），skill 会自动拉取该分组下所有流水线供用户选择，选中后拉取该流水线关联仓库的所有分支供用户选择，最终确认并运行。全程逐级点选，无需手动输入流水线名或分支名。当用户说"运行流水线"、"触发流水线"、"执行流水线"、"跑流水线"、"部署"、"pipeline run"、提到 test/beta/prod 分组等场景时使用本 skill。强烈建议在涉及 yunxiao 流水线运行任务的场景中都优先触发本 skill。
license: MIT
metadata:
  author: syf
  version: "1.1"
---

# glodrapay-pipeline-run

GlodraPay yunxiao 流水线运行助手。按分组筛选流水线、逐级选择、确认运行。

## 触发场景

- 用户要求运行/触发/执行某个流水线
- 用户提到"跑流水线"、"部署XX环境"、"触发构建"
- 用户说"用XX分支运行/跑一下"

## 执行流程

每次运行本 skill 必须严格按以下步骤顺序执行，不可跳步。

### 第一步：获取组织信息 & 收集分组

1. 调用 `yunxiao_get_current_organization_info` 获取当前组织 ID（`lastOrganization` 字段）

2. 使用 `question` 工具让用户选择分组：

   - **test** — 测试环境分组
   - **beta** — 预发环境分组
   - **prod** — 生产环境分组

### 第二步：列出该分组下的所有流水线

1. 调用 `yunxiao_list_pipelines`，`perPage` 设为 30，循环翻页直到获取全部流水线

2. 对返回的每条流水线，调用 `yunxiao_get_pipeline` 获取详细信息，读取其中的 `groupId` 字段

3. **仅保留 `groupId` 与用户选择的分组匹配的流水线**：
   - 分组名与 `groupId` 的映射关系需要在实际调用中确认——不同组织的 groupId 值不同
   - 关键：通过 `get_pipeline` 返回的 `groupId` 来判断归属，不是通过流水线名称

4. 如果匹配的流水线数量：
   - **多条**：使用 `question` 工具列出所有匹配的流水线名称（单选），让用户选择一条
   - **一条**：直接选中该流水线，进入第三步
   - **零条**：告知用户该分组下无流水线，结束流程

### 第三步：列出该流水线的所有分支

1. 从上一步获取的流水线详情中，读取 `sources[0].data.repo`，即代码仓库地址
   - 示例：`https://codeup.aliyun.com/687ef75e94fa1f53c813c620/frontend/glodrapay-web-admin-mso.git`
   - 从中提取仓库路径（`organizationId` 之后的部分）：将 `/` 替换为 `%2F`，得到 `repositoryId`
   - 该示例的 `repositoryId` 为：`687ef75e94fa1f53c813c620%2Ffrontend%2Fglodrapay-web-admin-mso`

2. 调用 `yunxiao_list_branches`，传入提取的 `organizationId` 和 `repositoryId`，`perPage` 设为 50，循环翻页获取全部分支

3. 如果分支数量：
   - **多条**：使用 `question` 工具列出分支名称（单选），让用户选择一条
   - **一条**：直接选中该分支
   - **零条**：告知用户该仓库无分支，结束流程

### 第四步：确认并运行

1. 向用户展示汇总信息：

   - 流水线名称
   - 所属分组（groupId）
   - 目标分支

2. 使用 `question` 工具询问用户是否确认运行

3. 用户确认后，调用 `yunxiao_create_pipeline_run`：

   | 参数 | 来源 |
   |------|------|
   | `organizationId` | 第一步自动获取 |
   | `pipelineId` | 第二步查到的流水线 ID |
   | `branch` | 第三步用户选择的分支名 |

4. 运行成功后，向用户报告结果（是否成功触发、运行 ID 等关键信息）

## 重要约定

### 中文沟通

本 skill 在与用户对话时使用中文，保持简洁专业的风格。

### groupId 匹配

- `groupId` 来自 `yunxiao_get_pipeline` 返回的流水线详情，是一个数字
- 匹配时需要先了解该组织的分组映射——可能通过 `get_pipeline` 多次调用建立分组名到 groupId 的映射表
- 如果在实践中发现不同分组有相同的 groupId 值模式，将其记录为已知映射

### 代码仓库提取

- 从 `sources[0].data.repo` 提取仓库路径
- URL 格式：`https://codeup.aliyun.com/{orgId}/{repoPath}.git`
- `repositoryId` = `{orgId}%2F{repoPath}`（将 `/` URL 编码为 `%2F`）
- 如果 `sources` 数组为空或无 `repo` 字段，告知用户无法获取分支列表，改为让用户手动输入分支名

### 分页处理

- `list_pipelines` 和 `list_branches` 都可能超过一页，需要循环翻页
- 翻页条件：当前页数据量等于 `perPage` 且 `pagination.total` 大于已获取数量

## 护栏

- 不跳过确认环节——必须向用户展示流水线和分支信息并获得确认后才能运行
- 不在未确认的情况下运行流水线
- 不主动修改流水线配置
- 如果流水线已在运行中，提醒用户并询问是否仍要触发
- 不猜测 groupId 映射——必须通过实际 API 返回的数据来判断
