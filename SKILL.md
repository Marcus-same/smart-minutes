---
name: smart-minutes
description: |
  飞书智能会议纪要。发送妙记链接自动生成结构化纪要文档。
  产出：SVG富卡片速览画板、核心结论、议题章节、决议、待办。
  触发：飞书妙记、会议纪要、智能纪要、妙记链接、feishu.cn/minutes/
version: "3.0.0"
---

# 飞书智能纪要

从妙记链接生成结构化纪要飞书文档。

## 文档结构（固定顺序）

```
1. 会议信息头部（blockquote：主题、时间、参会人 @cite）
2. 核心结论（3-5条，callout 高亮 ol）
3. 📊 会议速览（SVG 画板 — 富卡片式议题总览）
4. 正文（按议题分组，每议题一章）
5. ✅ 待办事项（checkbox 列表）
6. 🔗 原妙记链接（url-preview）
```

## 核心原则

1. **独立分析**：基于逐字稿提炼，不搬运 AI 总结
2. **SVG 画板**：速览图用 `<whiteboard type="svg">` 富卡片，不用 Mermaid mindmap
3. **富 block**：callout/grid/checkbox/table/whiteboard 替代纯文本
4. **信息密度**：连续纯文本 ≤3段，富 block 密度 ≥40%，元素类型 ≥3种
5. **简洁去 AI**：精准可执行，不堆黑话

## 工作流（4步）

### Step 1：提取数据

从妙记链接提取 `minute_token`（正则 `/minutes/([a-zA-Z0-9]+)`），并行拉取：

```bash
lark-cli vc +notes --minute-tokens "<minute_token>" --format json
lark-cli vc +search --meeting-ids "<meeting_id>" --format json
```

产物选择：
- 内容总结 → transcript（逐字稿），不用 AI summary
- 待办 → 参考 AI todo + transcript 补充
- 章节 → 参考 AI chapter + transcript 细化
- 元信息 → vc meeting 数据

拉取逐字稿：
```bash
lark-cli docs +fetch --api-version v2 --doc "<verbatim_doc_token>" --doc-format markdown
```

### Step 2：分析

基于逐字稿独立提炼：

**核心结论（3-5条）**：每条1-2句，直击决策结果。front-load，按影响面排序。

**议题分组**：按业务领域归类（人事/招聘/培训/薪酬/制度等），每类 3-5 条要点 + 决议。

**决议记录**：已明确的决策、投票结果、拍板事项。融入正文对应议题中，不单独成章。

**待办事项**：任务 + @负责人 + 截止时间。逐字稿未明确负责人的标注"待确认"，不猜测。按优先级排序。

### Step 3：创建文档骨架

```bash
lark-cli docs +create --api-version v2 --content @skeleton.xml
```

`skeleton.xml` 模板见 `references/layout-templates.md` 第 1 节。

骨架包含：标题、会议信息 blockquote、免责声明、核心结论占位、📊 会议速览 heading（等 SVG 插入）、各议题 h2 标题、待办占位、妙记链接。

保存返回的 `document_id` 和各 block 的 `block_id`（关键：速览 heading block_id 用于后续 SVG 插入）。

### Step 4：精修 + 画板

用 `docs +update` 的 `block_insert_after` 分章节写入。

**4a. 填充核心结论**

找到核心结论占位 block，`block_insert_after` 写入 callout + ol。

**4b. 插入速览画板（必选，在 📊 会议速览 heading 之后）**

```bash
lark-cli docs +update --api-version v2 --doc "<doc_id>" \
  --command block_insert_after \
  --block-id "<speedview_heading_block_id>" \
  --content '<whiteboard type="svg">...完整 SVG...</whiteboard>'
```

SVG 画板按议题数自适应布局，详细模板和色板见 `references/whiteboard-design.md`。

**4c. 按章节写入正文**

每章模式（决议融入各章节）：
```xml
<callout emoji="🎯" background-color="light-blue">
  <p><b>本章要点：</b>{一句话概括}</p>
</callout>

<grid>
  <column width-ratio="0.5">
    <h3>讨论内容</h3>
    <ul><li>...</li></ul>
  </column>
  <column width-ratio="0.5">
    <h3>决议与下一步</h3>
    <ul><li>...</li></ul>
  </column>
</grid>
```

**4d. 待办事项**

```xml
<checkbox done="false">{任务描述} — @{负责人} ⏰ {截止日期}</checkbox>
```

用 `lark-contact` 解析 user-id 写 `<cite type="user">`，无则用纯文本 @姓名。

**4e. 章节内画板（按需）**

| 内容特征 | 画板类型 |
|---------|---------|
| 多步流程 | `<whiteboard type="mermaid">flowchart` |
| 时间阶段 | `<whiteboard type="mermaid">timeline` |
| 方案对比 | `<whiteboard type="svg">` 双栏卡片 |
| 因果归因 | `<whiteboard type="svg">` 鱼骨图 |
| 占比分布 | `<whiteboard type="mermaid">pie` |

## 格式规范

### XML 快速参考

| 标签 | 用途 | 关键属性 |
|------|------|---------|
| `<callout>` | 高亮框 | `emoji`, `background-color`, `border-color` |
| `<grid>` + `<column>` | 分栏 | `width-ratio`（合计=1） |
| `<checkbox>` | 待办 | `done="true\|false"` |
| `<whiteboard>` | 画板 | `type="mermaid\|svg"` |
| `<table>` | 表格 | 标准 HTML table |
| `<hr/>` | 章节分隔 | — |
| `<blockquote>` | 引用/免责 | — |
| `<cite type="user">` | @人 | `user-id` |

转义：标签本身不转义，仅内部文本 `&` → `&amp;`，`<` → `&lt;`，`>` → `&gt;`。

### 颜色语义

| 语义 | callout 背景 | 文字色 | emoji |
|------|-------------|--------|-------|
| 信息/结论 | `light-blue` | `blue` | 💡 |
| 成功/推荐 | `light-green` | `green` | ✅ |
| 警告/风险 | `light-yellow` | `yellow` | ⚠️ |
| 高优/紧急 | `light-red` | `red` | ❗ |

表头 `background-color="light-gray"`。关键指标 `<span text-color="green/red">` + 方向标记 ↑↓。

### 丰富度自检

| 指标 | 达标标准 |
|------|---------|
| 富 block 密度 | ≥40% |
| 元素多样性 | ≥3 种 block 类型 |
| 连续纯文本 | ≤3 段连续 `<p>` |
| 章节丰富度 | 每个 h2 ≥1 个非纯文本 block |
| 开头 callout | 必须有（核心结论） |
| 章节分隔 | 不同议题间有 `<hr/>` |
| 画板存在 | 至少 1 个 `<whiteboard>` |

## 关键约束

1. 内容总结基于 transcript，AI summary 仅辅助
2. 速览图用 `<whiteboard type="svg">` 富卡片，不用 Mermaid mindmap
3. Windows 含中文命令先 `chcp 65001`
4. 逐字稿 >60 分钟按 chapter 分段分析后拼接
5. 待办负责人未明确标注"待确认"
6. 决议融入各议题章节，不再设独立「决议记录」章节
7. 直接输出文档链接，不额外说明

## 参考文件

- `references/layout-templates.md` — 骨架 XML + 章节模板 + 待办模板
- `references/whiteboard-design.md` — SVG 画板模板与色板
- `references/phone-setup.md` — 手机物理按键配置指南
- `lark-shared/SKILL.md` — 认证
- `lark-vc/references/vc-domain-boundaries.md` — 产物选择
