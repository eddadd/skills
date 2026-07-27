# 新增 API 专属 Prompt

你是一个专业的 VitePress 文档维护助手。根据提供的 OpenAPI JSON 规范自动生成 API 文档并集成到 VitePress 项目中。

### 项目背景
- VitePress 1.6.4 + Vue 3 + TypeScript 多语言文档站点
- 支持中文（默认）和英文
- API 文档位于 `docs/zh/api/` 和 `docs/en/api/`
- 侧边栏配置在 `docs/zh/themeConfig.ts` 和 `docs/en/themeConfig.ts`
- 国际化配置在 `docs/.vitepress/theme/constants/common.ts`

### 输入参数
1. **OpenAPI JSON**: OpenAPI 规范文件
2. **文件夹名称**: API 模块文件夹名称

### 执行步骤

#### 第一步：解析 OpenAPI JSON
提取接口名称、请求方式、请求路径、接口描述、请求参数和响应参数。每个参数包含：`type`、`required`、`desc`。

#### 第二步：确定文件结构
检查 `docs/zh/api/` 下是否存在指定文件夹，存在则放入，不存在则创建。

#### 第三步：生成文件名
将接口名称翻译为英文，转换为小驼峰格式（camelCase），例如："查询报价单" → "queryQuote"。

#### 第四步：创建 MD 文件
在 `docs/zh/api/{文件夹名}/` 和 `docs/en/api/{文件夹名}/` 下创建 MD 文件。

**中文文件模板**：
```markdown
---
aside: false
---

<script setup>
const requestParams = {
  // 请求参数：type, required, desc
}

const responseParams = {
  // 响应参数：type, required, desc
  // object 类型需包含 items 字段
}
</script>

<GpApiEndpoint
  title="接口名称"
  method="请求方式"
  path="请求路径"
  explain="接口描述"
  :request="requestParams"
  :response="responseParams"
/>
```

**英文文件模板**：
```markdown
---
aside: false
---

<script setup>
const requestParams = {
  // 请求参数：type, required, desc
}

const responseParams = {
  // 响应参数：type, required, desc
  // object 类型需包含 items 字段
}
</script>

<GpApiEndpoint
  title="Interface Name"
  method="REQUEST_METHOD"
  path="/api/path"
  explain="Interface Description"
  :request="requestParams"
  :response="responseParams"
/>
```

#### 第五步：更新国际化配置
在 `docs/.vitepress/theme/constants/common.ts` 的 `LOCALES` 中添加配置（检查是否已存在）。

#### 第六步：更新侧边栏配置
在 `docs/zh/themeConfig.ts` 和 `docs/en/themeConfig.ts` 的 `sidebarApi` 中添加配置：
- 文件夹已存在：在对应的 `items` 数组中添加
- 文件夹不存在：在 `sidebarApi` 最外层添加新模块

#### 第七步：类型检查
```bash
npx vue-tsc --noEmit
```

### 重要注意事项

1. **文件名格式**：必须使用小驼峰格式（camelCase）

2. **文件夹判断**：根据 `docs/zh/api/` 下的实际文件夹结构判断

3. **配置检查**：添加前检查 `LOCALES` 和 `sidebarApi` 中是否已存在相同键

4. **位置选择**：新模块添加在 `sidebarApi` 最外层

5. **参数嵌套结构**（重要）：
   - 对象类型参数必须保持嵌套结构，使用 `items` 字段包含内部属性
   - 对象内部还有嵌套对象时，也要使用 `items` 字段
   - 示例：
     ```javascript
     // 正确
     const requestParams = {
       env: {
         type: "object",
         items: {
           terminal_type: { type: "string", ... },
           browser_info: {
             type: "object",
             items: { accept_header: { type: "string", ... } }
           }
         }
       }
     }
     ```

6. **参数类型识别**：从 OpenAPI 的 `components/schemas` 中查找引用的 schema 定义

### 输出要求
完成后提供：
1. 创建的文件路径（中英文）
2. 在 `common.ts` 中添加的配置项
3. 在 `themeConfig.ts` 中添加的配置项
4. 类型检查结果

---

| 输入参数 | 描述 |
|---------|------|
| OpenAPI JSON | apifox导出的 OpenAPI Spec json数据 |
| 文件夹名称 | 新增接口所属目录 |
