# 新增地区 专属 Prompt

## 添加新地区到文档系统的提示词

### 需要提供的信息：
- **地区名**（中文）：[地区名]
- **参数文件列表**：[{ 文件名（中文）, 文件内容 }]
- **基础文档配置**：
  - `docsList` 数组
  - `modules` 数组（subItems 的 text 与参数文件名对应）

### 操作步骤：

#### 1. 更新国际化配置
在 `docs/.vitepress/theme/constants/common.ts` 中检查并添加参数文件名的国际化配置，不存在则添加到最上面。

#### 2. 创建 md 文件
在 `docs/zh/region/` 下新建文件夹（地区英文名），创建：
- **参数文件**：文件名=参数英文名（不要加地区名），内容=一级标题"地区名+文件名" + 提供的参数内容
- **overView.md**：使用模板填充 `docsList` 和 `modules`（subItems 的 text 用文件名）

#### 3. 创建英文版本
在 `docs/en/region/` 下创建相同结构，标题翻译为英文。

#### 4. 更新配置文件
- `docs/zh/themeConfig.ts`：在 `regionNavData` 和 `sidebarRegion` 添加新项（使用 `LOCALES[LANGUAGE.zh]`）
- `docs/en/themeConfig.ts`：同上（使用 `LOCALES[LANGUAGE.en]`）
- `docs/config.ts`：在 `sidebar` 添加 `/region/地区英文名/`
- `docs/en/config.ts`：在 `sidebar` 添加 `/en/region/地区英文名/`

### ⚠️ 注意事项：
1. 参数文件名只包含参数名称，**不要在前面加地区名**（地区名只在内容的一级标题中出现）
2. 参数文件内容必须使用提供的内容，**不要用其他内容**
3. `modules` 中 `subItems` 的 `text` 使用文件名（英文）
4. 地区英文名保持一致（文件夹名、link、sidebar 键名）
5. 完成后运行 `npx vue-tsc --noEmit` 检查

---

| 需要提供的信息 | 描述 |
|--------------|------|
| 地区名（中文） | 要新增的地区名（例：土耳其） |
| 参数文件列表 | 即该地区参数示例，和apifox保持一致（例：[{ 支付参数, apifox内容 }]） |
| 基础文档配置 | 即地区下开始对接的基础文档，基础文档和接口文档需要自己传递数据，需和项目中保持一致，参考：docs/.vitepress/theme/constants/common.ts的国际化 |
