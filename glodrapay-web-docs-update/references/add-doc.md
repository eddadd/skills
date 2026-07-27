# 新增文档 专属 Prompt

# VitePress 文档添加操作提示词

## 输入信息

- 中文标题：[标题]
- 文档内容：[Markdown内容]
- 文件夹名称：[可选，默认：basicDocs]

## 执行步骤

### 1. 生成文件名
根据中文标题翻译成英文，转换为小驼峰格式。如标题过长可采用简写。

### 2. 创建中文文档
在文档内容顶部添加一级标题（中文标题），保存到 `docs/zh/guide/[文件夹名]/[文件名].md`。如无文件夹名则直接放在 `docs/zh/guide/` 下。

### 3. 创建英文文档
将中文文档翻译为英文，顶部添加一级标题（英文标题），保存到 `docs/en/guide/[文件夹名]/[文件名].md`。

### 4. 更新 common.ts
编辑 `docs/.vitepress/theme/constants/common.ts`，在 `LOCALES` 中添加：

```typescript
// 英文
[文件名]: "[英文标题]",
// 中文
[文件名]: "[中文标题]",
```

如键已存在则跳过。

### 5. 更新 zh/themeConfig.ts
编辑 `docs/zh/themeConfig.ts`，在 `sidebarDocs` 中添加：

**直接在 guide 下：**
```typescript
{
  text: LOCALES[LANGUAGE.zh].[文件名],
  link: "[文件名]",
},
```

**在文件夹下：**
```typescript
{
  text: LOCALES[LANGUAGE.zh].[文件名],
  link: "[文件夹名]/[文件名]",
},
```

**如文件夹配置不存在，创建新项：**
```typescript
{
  text: LOCALES[LANGUAGE.zh].[文件夹名],
  items: [
    {
      text: LOCALES[LANGUAGE.zh].[文件名],
      link: "[文件夹名]/[文件名]",
    },
  ],
},
```

### 6. 更新 en/themeConfig.ts
编辑 `docs/en/themeConfig.ts`，在 `sidebarDocs` 中执行与步骤4相同的操作，使用 `LANGUAGE.en`。

## 示例

输入：中文标题="支付接口和请款接口区别"，文件夹="basicDocs"

AI生成文件名：paymentAndCaptureApiDifference

**中文文档：** `docs/zh/guide/basicDocs/paymentAndCaptureApiDifference.md`，标题 `# 支付接口和请款接口区别`

**英文文档：** `docs/en/guide/basicDocs/paymentAndCaptureApiDifference.md`，标题 `# Payment and Capture API Difference`

**common.ts：**
```typescript
paymentAndCaptureApiDifference: "Payment and Capture API Difference", // 英文
paymentAndCaptureApiDifference: "支付接口和请款接口区别", // 中文
```

**zh/themeConfig.ts：**
```typescript
{
  text: LOCALES[LANGUAGE.zh].paymentAndCaptureApiDifference,
  link: "basicDocs/paymentAndCaptureApiDifference",
},
```

**en/themeConfig.ts：**
```typescript
{
  text: LOCALES[LANGUAGE.en].paymentAndCaptureApiDifference,
  link: "basicDocs/paymentAndCaptureApiDifference",
},
```

## 注意事项

- 文件名使用小驼峰格式
- 键名与文件名保持一致（不含.md）
- 检查配置是否已存在，避免重复
- 侧边栏位置根据文档逻辑顺序放置

---

| 输入信息 | 描述 |
|---------|------|
| 中文标题 | 该文档的中文标题 |
| 文档内容 | 和apifox上内容一致（markdown） |
| 文件夹名称 | [可选] 新增文档所属目录，不填默认最外层 |
