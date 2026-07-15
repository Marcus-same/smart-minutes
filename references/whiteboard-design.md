---
description: 智能纪要画板 — SVG 富卡片速览 + 章节按需图形
---

# 画板设计规范

## 速览图：SVG 富卡片

会议速览用 `<whiteboard type="svg">` 生成卡片式议题总览，不用 Mermaid mindmap。

### 设计规则
- 每议题一张彩色卡片，图标标题 + 2-4条关键结论（每条≤12字）
- 不同议题不同底色
- 底部横排待办卡片
- 决议项用辅助灰色标注

### 色板（商务低饱和）

| 序号 | 底色 | 标题栏 | 边框 |
|-----|------|--------|------|
| 1 | `#EDF2F7` | `#4A6FA5` | `#4A6FA5` |
| 2 | `#E8EDF3` | `#5A7B9A` | `#5A7B9A` |
| 3 | `#F0F0F0` | `#8895A7` | `#8895A7` |

统一：borderWidth=2, borderRadius=8, 正文色=#1A202C, 辅助色=#718BAE

### 模板：3 议题 + 待办横排

```xml
<whiteboard type="svg">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 480">
  <!-- 标题 -->
  <text x="500" y="32" text-anchor="middle" font-size="22" fill="#1A202C" font-weight="bold">{会议名称}</text>
  <text x="500" y="54" text-anchor="middle" font-size="12" fill="#718BAE">{日期} · {时长} · {N}人参会</text>

  <!-- 卡片1 -->
  <g transform="translate(20, 76)">
    <rect x="0" y="0" width="300" height="160" rx="8" fill="#EDF2F7" stroke="#4A6FA5" stroke-width="2"/>
    <rect x="0" y="0" width="300" height="34" rx="8" fill="#4A6FA5"/>
    <rect x="0" y="16" width="300" height="18" fill="#4A6FA5"/>
    <text x="16" y="23" font-size="14" fill="#FFFFFF" font-weight="bold">{图标} {类别}</text>
    <text x="16" y="58" font-size="12" fill="#1A202C">• {关键点1，≤15字}</text>
    <text x="16" y="78" font-size="12" fill="#1A202C">• {关键点2，≤15字}</text>
    <text x="16" y="98" font-size="12" fill="#1A202C">• {关键点3，≤15字}</text>
    <text x="16" y="124" font-size="11" fill="#718BAE">  决议：{结论，≤20字}</text>
  </g>

  <!-- 卡片2（色板2） -->
  <g transform="translate(340, 76)">
    <rect x="0" y="0" width="300" height="160" rx="8" fill="#E8EDF3" stroke="#5A7B9A" stroke-width="2"/>
    <rect x="0" y="0" width="300" height="34" rx="8" fill="#5A7B9A"/>
    <rect x="0" y="16" width="300" height="18" fill="#5A7B9A"/>
    <text x="16" y="23" font-size="14" fill="#FFFFFF" font-weight="bold">{图标} {类别}</text>
    <text x="16" y="58" font-size="12" fill="#1A202C">• {关键点1}</text>
    <text x="16" y="78" font-size="12" fill="#1A202C">• {关键点2}</text>
    <text x="16" y="98" font-size="12" fill="#1A202C">• {关键点3}</text>
    <text x="16" y="124" font-size="11" fill="#718BAE">  决议：{结论}</text>
  </g>

  <!-- 卡片3（色板3） -->
  <g transform="translate(660, 76)">
    <rect x="0" y="0" width="300" height="160" rx="8" fill="#F0F0F0" stroke="#8895A7" stroke-width="2"/>
    <rect x="0" y="0" width="300" height="34" rx="8" fill="#8895A7"/>
    <rect x="0" y="16" width="300" height="18" fill="#8895A7"/>
    <text x="16" y="23" font-size="14" fill="#FFFFFF" font-weight="bold">{图标} {类别}</text>
    <text x="16" y="58" font-size="12" fill="#1A202C">• {关键点1}</text>
    <text x="16" y="78" font-size="12" fill="#1A202C">• {关键点2}</text>
    <text x="16" y="98" font-size="12" fill="#1A202C">• {关键点3}</text>
    <text x="16" y="124" font-size="11" fill="#718BAE">  决议：{结论}</text>
  </g>

  <!-- 待办横排 -->
  <g transform="translate(20, 256)">
    <rect x="0" y="0" width="940" height="100" rx="8" fill="#D4E0ED" stroke="#4A6FA5" stroke-width="2"/>
    <text x="20" y="26" font-size="14" fill="#1A202C" font-weight="bold">待办事项</text>
    <text x="20" y="52" font-size="12" fill="#1A202C">• {事项1} @负责人 {截止日}</text>
    <text x="20" y="74" font-size="12" fill="#1A202C">• {事项2} @负责人 {截止日}</text>
    <text x="470" y="52" font-size="12" fill="#1A202C">• {事项3} @负责人 {截止日}</text>
    <text x="470" y="74" font-size="12" fill="#1A202C">• {事项4} @负责人 {截止日}</text>
  </g>
</svg>
</whiteboard>
```

### 待办子行布局

待办横排区域显示 4 条待办（如不足4条则只显示实际数量）：
- 左列（x=20）：事项1(y=52)、事项2(y=74)
- 右列（x=470）：事项3(y=52)、事项4(y=74)

### 布局自适应

- **2-3 议题** → 卡片~300px 宽，一行（参考 3 议题模板）
- **4-5 议题** → 卡片~220px 宽，分两行
  - 第一行 3 张 (y=76)，第二行 2 张 (y=256)
  - viewBox 高度 = 480
- **6+ 议题** → 卡片~300px 宽，分两行
  - 第一行 3 张 (y=76)，第二行 3 张 (y=256)
  - viewBox 高度 = 500

总高估算：60(标题) + N行×180 + 120(待办) + 20(padding)

### 4-5 议题第二行卡片位置

```xml
<!-- 第二行：卡片4 -->
<g transform="translate(20, 256)">
  <rect x="0" y="0" width="220" height="150" rx="8" fill="#EDF2F7" stroke="#4A6FA5" stroke-width="2"/>
  ...
</g>
<!-- 卡片5 -->
<g transform="translate(260, 256)">
  <rect x="0" y="0" width="220" height="150" rx="8" fill="#E8EDF3" stroke="#5A7B9A" stroke-width="2"/>
  ...
</g>
<!-- 待办 y 下移 -->
<g transform="translate(20, 426)">
```

## 章节按需图形

### Mermaid（简单流程/时序/饼/甘特）

```xml
<whiteboard type="mermaid">
flowchart LR
  A[开始] --> B{判断} -->|是| C[执行]
</whiteboard>
```

### SVG（复杂对比/架构/鱼骨/金字塔）

```xml
<whiteboard type="svg">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 900 400">
  <!-- 色板、卡片式规范同上 -->
</svg>
</whiteboard>
```

## 禁止

- 禁止 `radialGradient` / `filter` / `clipPath` / `mask` / `pattern`
- 文字用 `<text>` 不用 `<path>`
- 连线正交折线不用斜线
