# DocxXML 布局模板

各会议类型和场景的 XML 骨架与组件模板。

## 1. 通用 skeleton.xml（创建文档骨架）

```xml
<title>智能纪要：{会议名称} {日期}</title>

<!-- 1. 会议信息头部 -->
<blockquote>
  <p>会议主题：{会议主题}</p>
  <p>会议时间：{日期}（{星期}） {开始时间} - {结束时间}（GMT+08 中国标准时间）</p>
  <p>参会人：{姓名1}<cite type="user" user-id="{user_id1}" user-name="{姓名1}"></cite>{姓名2}<cite type="user" user-id="{user_id2}" user-name="{姓名2}"></cite>……</p>
</blockquote>

<blockquote>
  <p>智能会议纪要由 AI 生成，可能不准确，请谨慎甄别后使用</p>
</blockquote>

<!-- 2. 核心结论占位 -->
<h2>💡 核心结论</h2>
<p>占位</p>

<!-- 3. 会议速览（Step 4 插入 SVG 画板）-->
<h2>📊 会议速览</h2>
<p>占位</p>

<!-- 4. 正文：议题章节 -->
<h2>{议题章节1}</h2>
<p>占位</p>

<h2>{议题章节2}</h2>
<p>占位>

<h2>{议题章节3}</h2>
<p>占位</p>

<h2>{议题章节4}</h2>
<p>占位</p>

<hr/>

<!-- 5. 待办事项 -->
<h2>✅ 待办事项</h2>
<p>占位</p>

<hr/>

<!-- 6. 原妙记链接 -->
<p>🔗 <a type="url-preview" href="{妙记原链接}">妙记原链接</a></p>
```

### 使用方式

```bash
lark-cli docs +create --api-version v2 --parent-position my_library --content @skeleton.xml
```

返回 JSON 中记录 `document_id` 和各 block 的 `block_id`。后续用 `block_insert_after` 替换各 `p>占位</p>`。

## 2. 核心结论填充（替换 💡 核心结论 后的占位 p）

```xml
<callout emoji="💡" background-color="light-blue" border-color="blue">
  <ol>
    <li seq="auto">{结论1 — 1-2句，直击决策结果}</li>
    <li seq="auto">{结论2}</li>
    <li seq="auto">{结论3}</li>
    <li seq="auto">{结论4}</li>
    <li seq="auto">{结论5}</li>
  </ol>
</callout>
```

3-5 条，按影响面排序。

## 3. 章节模板（每种会议类型通用）

每章固定在对应 h2 heading 后写入：

```xml
<callout emoji="🎯" background-color="light-blue">
  <p><b>本章要点：</b>{一句话概括本章核心}</p>
</callout>

<grid>
  <column width-ratio="0.5">
    <h3>讨论内容</h3>
    <ul>
      <li>{讨论要点1}</li>
      <li>{讨论要点2}</li>
      <li>{讨论要点3}</li>
      <li>{讨论要点4}</li>
    </ul>
  </column>
  <column width-ratio="0.5">
    <h3>决议与下一步</h3>
    <ul>
      <li>{决议1}</li>
      <li>{决议2}</li>
    </ul>
  </column>
</grid>
```

**原则**：
- 决议融入各议题章节，不设独立「决议记录」章节
- 每章 3-5 条讨论要点 + 1-3 条决议
- 讨论内容侧重事实和数据，决议侧重结论和行动

## 4. 待办事项填充（替换 ✅ 待办事项 后的占位 p）

```xml
<checkbox done="false">{任务描述} — @{负责人} ⏰ {截止日期}</checkbox>
<checkbox done="false">{任务描述} — @{负责人} ⏰ {截止日期}</checkbox>
<checkbox done="false">{任务描述} — @{负责人} ⏰ {截止日期}</checkbox>
```

**规则**：
- 有 `user_id` 的用 `<cite type="user" user-id="...">` 包 @姓名，无则纯文本
- 未明确负责人标注"待确认"
- 按优先级排列
- DONE 项放末尾，`done="true"`

## 5. 会议类型专属变体

### 面试

章节命名改为面试评估维度（如"技术能力评估""项目经验深挖""团队匹配度""综合素质"等）。

每章讨论内容侧重候选人回答和表现，决议侧重评分和面试官评价。

在核心结论 callout 中开头加一条录用建议：

```xml
<li seq="auto"><b>录用建议：{推荐/待定/不推荐}</b> — {一句话理由}</li>
```

速览 SVG 卡片配色改为录用信号：推荐=绿色系，待定=黄色系，不推荐=红色系。

### 培训

章节命名改为知识模块（如"核心概念""实操演示""案例分析""Q&A 互动"等）。

每章讨论内容改为「讲授内容」，决议改为「关键 Takeaway」。

Q&A 交互多的培训，在正文后加一个 Q&A 汇总 block：

```xml
<h2>🙋 Q&A 回顾</h2>
<callout emoji="🙋" background-color="light-blue">
  <p><b>Q1：{问题}</b></p>
  <p>A：{回答要点}</p>
  <hr/>
  <p><b>Q2：{问题}</b></p>
  <p>A：{回答要点}</p>
</callout>
```

### 项目同步

章节命名改为业务线/模块名（如"后端引擎""前端渲染""数据平台""质量保障"等）。

每章"决议与下一步"改为"本周 Milestone + Blockers"，状态用颜色信号：

```xml
<li><span color="green">✅ On Track</span> {事项}</li>
<li><span color="orange">⚠️ At Risk</span> {事项} — {风险说明}</li>
<li><span color="red">🔴 Blocked</span> {事项} — {阻塞原因}</li>
```

### 1v1 面谈 / 小范围讨论

如果只有 2-3 个议题、逐字稿偏短（<30分钟），章节结构不变但议题 ≤3 个，省略速览画板或改用 2 卡片布局。

## 6. 精简模板（< 15 分钟短会）

省略会议速览画板和章节细分，只保留核心：

```xml
<title>智能纪要：{会议名称} {日期}</title>

<blockquote>
  <p>会议主题：{会议主题}</p>
  <p>会议时间：{日期} {开始时间}-{结束时间}  参会人：{姓名1}、{姓名2}</p>
</blockquote>

<blockquote>
  <p>智能会议纪要由 AI 生成，可能不准确，请谨慎甄别后使用</p>
</blockquote>

<callout emoji="💡" background-color="light-blue" border-color="blue">
  <h2>核心结论</h2>
  <ol>
    <li seq="auto">{结论1}</li>
    <li seq="auto">{结论2}</li>
    <li seq="auto">{结论3}</li>
  </ol>
</callout>

<callout emoji="🎯" background-color="light-green" border-color="green">
  <h2>讨论与决议</h2>
  <p>{综合叙述}</p>
</callout>

<callout emoji="✅" background-color="light-yellow" border-color="orange">
  <h2>待办事项</h2>
  <checkbox done="false">{事项1} — @负责人 ⏰ 截止</checkbox>
  <checkbox done="false">{事项2} — @负责人 ⏰ 截止</checkbox>
</callout>

<hr/>
<p>🔗 <a type="url-preview" href="{妙记链接}">妙记原链接</a></p>
```

## 注意事项

1. `<whiteboard>` 标签在 `docs +create` 和 `docs +update` 时只能新增，不能修改已有画板
2. `<grid>` 内只能放 `<column>`，`<column>` 的 `width-ratio` 总和必须为 1
3. `<callout>` 内只能放文本、标题、列表、待办、引用，不能放表格或 grid
4. 表格 `<td>` 的 `background-color` 仅支持基础色名，不支持 `light-*` 变体
5. `<cite type="user">` 需带 `user-id` 属性实现 @人，user-id 用 `lark-contact` 解析
6. 妙记链接用 `<a type="url-preview">` 展示链接预览卡片
