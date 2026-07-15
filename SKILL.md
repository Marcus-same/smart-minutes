---
name: smart-minutes
description: |
  飞书智能会议纪要。发送妙记链接自动生成结构化纪要文档。
  产出：SVG富卡片速览画板、核心结论、议题章节、决议、待办。
  触发：飞书妙记、会议纪要、智能纪要、妙记链接、feishu.cn/minutes/
version: "3.1.0"
---

# 飞书智能纪要

从妙记链接生成结构化纪要飞书文档。

## 文档结构（固定顺序）

```
1. 会议信息头部（blockquote：主题、时间、参会人 @cite）
2. 核心结论（3-5条，callout 高亮 ol）
3. 📊 会议速览（SVG 画板 — 富卡片式议题总览；短会可降级为 callout 列表）
4. 正文（按议题分组，每议题一章）
5. ✅ 待办事项（checkbox 列表）
6. 🔗 原妙记链接（url-preview）
```

## 核心原则

1. **独立分析**：基于逐字稿提炼，不搬运 AI 总结
2. **SVG 画板**：速览图用 `<whiteboard type="svg">` 富卡片，不用 Mermaid mindmap；≤2 议题或 <15 分钟的短会可降级为 callout 列表
3. **富 block**：callout/grid/checkbox/table/whiteboard 替代纯文本
4. **信息密度**：连续纯文本 ≤3段，富 block 密度 ≥40%，元素类型 ≥3种
5. **简洁去 AI**：精准可执行，不堆黑话

## 工作流

### Step 0：前置检查

1. 从妙记链接提取 `minute_token`（正则 `/minutes/([a-zA-Z0-9]+)`）
2. 拉取妙记数据：
```bash
lark-cli vc +notes --minute-tokens "<minute_token>" --format json
```
3. 如果返回的 artifacts 为空或 transcript_file 不存在 → 妙记可能仍在转写中，告知用户等待转写完成后再试
4. 如果有 meeting_id（非必须），可额外拉取会议元信息丰富参会人数据：
```bash
lark-cli vc +search --meeting-ids "<meeting_id>" --format json
```

**产物优先级**：
- 内容总结 → **逐字稿（transcript）**，AI summary 仅辅助参考
- 逐字稿优先读取 `vc +notes` 已保存的本地文件 `minutes/<minute_token>/transcript.txt`
- 待办 → 参考 AI todo + transcript 补充
- 章节 → 参考 AI chapter + transcript 细化
- 元信息 → vc meeting 数据（如有）

### Step 1：分析

基于逐字稿独立提炼：

**核心结论（3-5条）**：每条1-2句，直击决策结果。front-load，按影响面排序。

**议题分组**：按业务领域归类（人事/招聘/培训/薪酬/制度等），每类 3-5 条要点 + 决议。

**决议记录**：已明确的决策、投票结果、拍板事项。融入正文对应议题中，不单独成章。

**待办事项**：任务 + @负责人 + 截止时间。逐字稿未明确负责人的标注"待确认"，不猜测。按优先级排序。

### Step 2：生成纪要文档

**推荐方式：overwrite 全文写入（一步到位）**

先创建空文档，再用 `overwrite` 一次性写入完整内容：

```bash
# 创建空文档
lark-cli docs +create --api-version v2 --content @skeleton.xml --as user
# 拿到 document_id 后，直接 overwrite 完整纪要
lark-cli docs +update --api-version v2 --doc "<doc_id>" --command overwrite --content @full_minutes.xml --as user
```

**备选方式：block_insert_after 逐章写入**

如需逐章替换，先用 `docs content get` 获取所有 block_id：
```bash
lark-cli docs content get --api-version v2 --doc "<doc_id>" --as user
```
然后定位各占位 block 的 ID，用 `block_insert_after` 替换。

文档骨架模板和章节模板见 `references/layout-templates.md`。

`skeleton.xml` 中 `--parent-position` 参数省略则默认存入我的空间，用户可自行移动。

### Step 3：插入速览画板

速览图使用 `<whiteboard type="svg">` 富卡片。

**根据议题数量选择布局**：
| 议题数 | 布局 | 详见 |
|--------|------|------|
| 3-5 个 | 横排卡片 + 底部待办 | `references/whiteboard-design.md` |
| 1-2 个 | 双栏卡片或单卡 | 简化模板 |
| 短会（<15分钟） | 省略画板，改用 callout 列表 | `references/layout-templates.md` 第 6 节 |

```bash
lark-cli docs +update --api-version v2 --doc "<doc_id>" \
  --command block_insert_after \
  --block-id "<speedview_heading_block_id>" \
  --content '<whiteboard type="svg">...完整 SVG...</whiteboard>'
```

**SVG 注意事项**：
- `<text>` 不支持自动换行，长文本需用多个 `<tspan>` 分行
- 不同议题用不同底色区分（色板见 whiteboard-design.md）
- SVG 画板创建后不可修改，需确保内容正确再提交

### Step 4：章节内画板（按需）

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
| 画板存在 | 至少 1 个 `<whiteboard>`（短会可豁免） |

## 关键约束

1. 内容总结基于 transcript，AI summary 仅辅助
2. 速览图用 `<whiteboard type="svg">` 富卡片，不用 Mermaid mindmap
3. Windows 含中文命令先 `chcp 65001`
4. 逐字稿 >60 分钟按 chapter 分段分析后拼接
5. 待办负责人未明确标注"待确认"
6. 决议融入各议题章节，不再设独立「决议记录」章节
7. 直接输出文档链接，不额外说明

## 异常处理

| 异常情况 | 处理方式 |
|---------|---------|
| 妙记仍在转写中（无 artifacts/transcript） | 告知用户等待转写完成，1-2 分钟后重试 |
| 逐字稿为空或 < 1 分钟 | 提示用户该妙记可能录制失败或内容过短 |
| `lark-contact` 查询用户失败 | 降级为纯文本 @姓名，不加 `<cite>` |
| API 返回权限错误 | 提示用户检查妙记分享权限或重新授权 |
| 逐字稿 > 60 分钟 | 按 chapter 分段分析后拼接结论 |

## 参考文件

- `references/layout-templates.md` — 骨架 XML + 章节模板 + 待办模板 + 会议类型变体
- `references/whiteboard-design.md` — SVG 画板模板与色板
- `references/phone-setup.md` — 📱 附录：手机一键启动妙记录音（iPhone/Android 配置指南）
- `lark-shared/SKILL.md` — 认证
- `lark-vc/references/vc-domain-boundaries.md` — 产物选择
