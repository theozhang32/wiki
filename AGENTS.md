# AGENTS.md — LLM Wiki 操作手册

> 通用版本，适用于 Claude Code、Codex CLI、Cursor、Windsurf、Gemini CLI、OpenClaw 等任何支持文件系统访问的 LLM Agent。
> 本文件是 Agent 的「运行时规则书」—— 每次会话开始前，Agent 必须先完整阅读本文件。

---

## 1. 系统概述

本仓库是一个 **LLM 维护的个人知识库（LLM Wiki）**，基于 [Andrej Karpathy 的 LLM Wiki 方法论](./schema/methodology.md)构建。

**核心理念**：
- 人类负责：筛选来源、提出好问题、判断矛盾
- Agent 负责：总结、交叉引用、归档、维护索引、标记矛盾
- 知识是「编译一次，持续复利」的产物，而非每次查询时重新推导

**与 RAG 的区别**：
| | RAG | LLM Wiki |
|---|---|---|
| 知识存储 | 原始文档分块 + 向量索引 | 结构化 Markdown 页面 |
| 综合时机 | 查询时实时推导 | Ingest 时预编译 |
| 交叉引用 | 每次重建 | 持续维护 |
| 矛盾发现 | 靠运气 | Ingest 时主动标记 |

---

## 2. 三层架构

```
raw/          ← 原始来源（不可变）—— Agent 只读，人类管理
wiki/         ← 知识库（Agent 全权维护）—— 人类阅读、评论、审阅
AGENTS.md     ← 本文件（Schema 规则书）—— 人类与 Agent 共同演化
```

**补充目录**（不打破三层架构）：

```
schema/       ← AGENTS.md 的补充材料（方法论、页面细则、模板参考）—— 详见 schema/README.md
```

### 2.1 raw/ — 原始来源层
- **规则**：Agent **绝对不可修改** raw/ 下的任何文件
- **用途**：存放原始文章、论文、剪藏网页、笔记、PDF 等
- **子目录**：
  - `raw/articles/` — 长篇文章和博客
  - `raw/papers/` — 学术论文和 PDF
  - `raw/books/` — 书籍章节或读书笔记
  - `raw/clippings/` — Obsidian Web Clipper 的收件箱（Ingest 时自动分类）
  - `raw/assets/` — 图片、图表、附件

### 2.2 wiki/ — 知识库层
- **规则**：Agent 拥有完整写入权限；人类可直接编辑，但必须在下次会话中告知 Agent 改了什么
- **子目录**（与 `type` frontmatter 一一对应）：
  - `wiki/sources/` — 每篇 raw 来源的摘要页（1 来源 = 1 页面，`type: source`）
  - `wiki/entities/` — 实体页：人物、组织、产品、地点（`type: entity`）
  - `wiki/concepts/` — 概念页：想法、框架、方法论、理论（`type: concept`）
  - `wiki/syntheses/` — 综合页：跨来源主题研究、演化中的论点（`type: synthesis`）
  - `wiki/comparisons/` — 对比页：两个或多个主体的并排分析（`type: comparison`）
  - `wiki/index.md` — 全 wiki 目录（每次 Ingest/Query 后更新）
  - `wiki/log.md` — 追加-only 操作日志（grep 可解析）
  - `wiki/overview.md` — 高层综合：当前论点、论点历史、开放问题、已知矛盾

### 2.3 schema/ — 规则书补充
- **规则**：Agent 可读写；人类与 Agent 共同演化
- **定位**：支撑 `AGENTS.md` 编写的补充材料，**不是第四层**。运行时约束以 `AGENTS.md` 为准；细则、长模板、外部参考放 `schema/`
- **子目录与文件**：
  - `schema/methodology.md` — Karpathy LLM Wiki 方法论
  - `schema/comparisons-slug.md` — Comparison 页 slug 命名细则
  - `schema/README.md` — 本目录索引与维护规则

---

## 3. Frontmatter 规范（强制）

**每篇 wiki 页面必须在文件开头包含 YAML frontmatter**，格式如下：

```yaml
---
type: source | entity | concept | synthesis | comparison
title: "Human Readable Title"
tags: [tag1, tag2]
sources: [source-slug.md]         # 支撑本页的 raw 来源文件名
created: YYYY-MM-DD
updated: YYYY-MM-DD
explored: false                   # 验证门：AI 永远设为 false，人类审阅后设为 true
confidence: high | medium | low | uncertain   # 内容可信度
---
```

### 字段说明

| 字段 | 必填 | 说明 |
|------|:----:|------|
| `type` | ✅ | 页面类型，必须与所在子目录一致：`sources/`→`source`，`entities/`→`entity`，`concepts/`→`concept`，`syntheses/`→`synthesis`，`comparisons/`→`comparison` |
| `title` | ✅ | 人类可读的页面标题（Title Case） |
| `tags` | ✅ | 主题标签数组，至少 1 个 |
| `sources` | ✅ | 关联的 raw 来源文件名（相对 raw/ 根目录） |
| `created` | ✅ | 页面创建日期 |
| `updated` | ✅ | 页面最后更新日期（知识内容变更时更新，非文件系统时间） |
| `explored` | ✅ | **验证门**。AI 创建/更新时永远设为 `false`。只有人类审阅后手动改为 `true`。防止盲目信任 AI 输出。 |
| `confidence` | ✅ | `high`（多来源交叉验证）、`medium`（单来源但合理）、`low`（推测性）、`uncertain`（矛盾或证据不足） |

### 新增字段规则
- **绝不预先臆测**新字段。只有当一次真实的 Lint 会话揭示需要时才添加。
- 任何字段添加都记录在 `## Schema History` 中。

---

## 4. 页面模板

### 4.1 Source 页（`wiki/sources/<kebab-case-slug>.md`）

```markdown
---
type: source
title: "来源标题"
tags: [topic1, topic2]
sources: [原始文件名.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
explored: false
confidence: medium
---

## Summary
2-4 段综合摘要，不是原文复制，而是提炼后的理解。

## Key Claims
- 核心主张 1（附上下文和证据）
- 核心主张 2

## Notable Quotes
> "来源中的直接引用，必须能在 raw/ 中 grep 到原文。"

## Counter-arguments
[什么在反驳本页的主张？即使来源本身没有提到，也要主动思考对立面。]

## Data gaps
[我们不知道什么？缺少什么数据？还需要哪些来源来补全？]

## Related Pages
- [[相关实体]]
- [[相关概念]]

## Sources
- `raw/articles/原始文件名.md`
```

### 4.2 Entity 页（`wiki/entities/<kebab-case-slug>.md`）

```markdown
---
type: entity
title: "实体名称"
tags: [person | org | product | place, topic-tag]
sources: [source1.md, source2.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
explored: false
confidence: high
---

一句话描述该实体是什么。

## What They Did / What It Is
跨所有来源的综合描述，不是简单罗列。

## Appearances
- [[Source Page 1]] — 在该来源中出现的上下文
- [[Source Page 2]] — 在该来源中出现的上下文

## Relations
- contradicts: [[Page]] — 一句话解释冲突
- supports: [[Page]] — 一句话解释支持关系
- depends_on: [[Page]] — 一句话解释依赖关系
- evolved_into: [[Page]] — 一句话解释演化关系
```

### 4.3 Concept 页（`wiki/concepts/<kebab-case-slug>.md`）

```markdown
---
type: concept
title: "概念名称"
tags: [framework | methodology | idea | theory, topic-tag]
sources: [source1.md, source2.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
explored: false
confidence: medium
---

一句话定义。

## Description
跨来源的综合描述。

## Key Properties / Variants
- 关键属性或变体 1
- 关键属性或变体 2

## Appears In
- [[Source Page]] — 如何被使用
- [[Entity Page]] — 与实体的关系

## See Also
- [[Related Concept]]

## Relations
- contradicts: [[Page]] — 一句话解释
- supports: [[Page]] — 一句话解释
```

### 4.4 Comparison 页（`wiki/comparisons/<slug-a>-vs-<slug-b>.md`）

> Slug 细则见 [schema/comparisons-slug.md](./schema/comparisons-slug.md)。参考 [Pratiyush/llm-wiki](https://github.com/Pratiyush/llm-wiki) 的字母序 `-vs-` 规范与 [MLMario/wiki-llm](https://github.com/MLMario/wiki-llm) 的 comparisons 目录实践。

```markdown
---
type: comparison
title: "A vs B"
tags: [comparison, topic-tag]
sources: [source1.md, source2.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
explored: false
confidence: medium
---

## Summary
一句话说明对比目的与当前结论倾向。

## Subjects
- [[Entity or Concept A]] — 在本对比中的角色
- [[Entity or Concept B]] — 在本对比中的角色

## Comparison

| Dimension | A | B |
|-----------|---|---|
| 维度 1 | ... | ... |
| 维度 2 | ... | ... |

## Key Differences
- 差异 1（附来源）
- 差异 2

## When to Choose Which
- 选 A 的场景
- 选 B 的场景

## Counter-arguments
[对比框架本身有何局限？是否遗漏重要维度？]

## Data gaps
[缺少哪些并排数据？还需要哪些来源？]

## Related Pages
- [[相关概念]]
- [[相关实体]]
```

**Slug 速查**：
- 二元对比：`<slug-a>-vs-<slug-b>.md`，两侧 slug **字母序**排列（保证每对只有一个规范文件）
- 多元对比（3+）：描述性 kebab-case，如 `vector-database-landscape.md`（不用 `a-vs-b-vs-c`）

### 4.5 Synthesis 页（`wiki/syntheses/<kebab-case-slug>.md`）

```markdown
---
type: synthesis
title: "综合标题"
tags: [topic1, topic2]
sources: [source1.md, source2.md, source3.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
explored: false
confidence: medium
---

## Summary
跨来源的综合观点。

## Key Findings
- 发现 1
- 发现 2

## Contradictions
> [!contradiction] Source A vs Source B
> [[source-a]] 主张 X，但 [[source-b]] 主张 Y。
> Resolution: {已声明的立场，或 "unresolved - flagged for review"}

## Related Pages
- [[相关概念]]
- [[相关实体]]
```

### 4.6 Overview 页（`wiki/overview.md`）—— 最重要的文件

```markdown
## Current Thesis
基于所有已摄入来源，你对该领域的当前立场。
如果还没有立场，写：「尚无明确立场 —— 将在摄入 5-10 篇来源后更新。」

## Thesis History
<!-- 每次 Current Thesis 变化时，把旧版本移到这里 -->
<!-- - YYYY-MM-DD — 先前立场，什么改变了它 -->

## Open Questions
<!-- - 问题 — 为什么重要 -->

## Known Contradictions
<!-- - [Source A] vs [Source B] 关于主张 X — open | investigating | resolved | accepted-tension -->

## Things That Changed My Mind
<!-- - 先前信念 → 修正后信念 — 触发更新的来源 -->

## Key Hubs
<!-- 每次 Lint 后更新。链接最多的 5-10 个页面。 -->
```

> **关键原则**：如果 `Current Thesis` 从未被修订过，说明你只是在归档，而不是在思考。知识库的价值在于观点随证据演化。

---

## 5. 核心工作流

### 5.1 INGEST — 摄入新来源

**触发方式**：用户说「请摄入 `raw/文件名`」或「处理这篇来源」

**执行步骤**（严格按顺序）：

1. **读取来源**：完整阅读 `raw/` 下的目标文件
2. **讨论框架**（默认执行）：向用户总结关键要点，说明你选择强调什么以及为什么，询问用户的框架是否正确。只有当用户明确说「skip discuss」或「batch-ingest」时才跳过。
3. **来源保真检查**：
   - 每个数字、日期、直接引用必须在写入前在 raw 文件中定位（用 grep 或读取）
   - 精确按原样写入 —— 来源写「42K」，你就写「42K」，不要改成「42,000」
   - 如果无法定位某个值，不要写精确形式；要么删除，要么用不精确的表述
4. **写 Source 页**：在 `wiki/sources/` 创建/更新摘要页
5. **提取实体**：列出所有提到的人、组织、产品、地点；为每个创建或更新 `wiki/entities/` 页面
6. **提取概念**：列出所有想法、框架、主题；为每个创建或更新 `wiki/concepts/` 页面
7. **矛盾检测**：将新来源的主张与现有 wiki 页面逐条对比：
   - 如果是时间更新（v1.0 → v2.0）→ 正常更新，旧版本标记 `Status: Outdated`
   - 如果是新信息 → 正常整合
   - 如果是真实矛盾 → 在双方页面添加 `## Contradictions` 节，状态设为 `open`
   - **绝不静默覆盖**矛盾信息
8. **Bias Check**：每个 concept/source/synthesis/comparison 页面必须包含：
   - `## Counter-arguments` — 什么在反驳本页主张
   - `## Data gaps` — 我们不知道什么
9. **更新 Overview**：如果新来源显著改变了综合观点，修订 `wiki/overview.md`
10. **更新 Index**：在 `wiki/index.md` 中添加/更新所有触及页面的条目
11. **追加 Log**：在 `wiki/log.md` 末尾添加：
    ```markdown
    ## [YYYY-MM-DD] ingest | <来源标题>
    - Disposition: New | Update | Disputed | No material
    - Raw: raw/<topic>/<filename>
    - Updated: <级联更新的页面标题>
    ```
12. **会话结束验证**：确认每个新创建/修改的页面都在 `wiki/index.md` 中有一行摘要，且 frontmatter 完整

**预期范围**：每篇来源触及 10-15 个 wiki 文件。

**页面创建阈值**：
- 某个主题在 **2+ 来源**中出现 → 创建完整页面
- 某个主题只在 **1 个来源**中出现 → 创建 stub 页面（frontmatter + 一句话定义 + 来源链接）
- **绝不**留下指向不存在的 `[[wikilink]]`

---

### 5.2 QUERY — 查询知识库

**触发方式**：用户提出任何问题

**执行步骤**：

1. **读取 Index**：先读 `wiki/index.md` 定位候选页面
2. **全文搜索**：用主题的关键术语**及其同义词**搜索 wiki/，不要仅依赖 Index
3. **读取页面**：打开相关页面，深入阅读
4. **综合答案**：
   - 优先使用 wiki 内容，而非自身训练知识
   - 用 `[[Page Title]]` 做内联引用
   - 在对话中输出答案
5. **默认归档**：综合答案后，询问用户是否保存为 wiki 页面。**默认归档**，除非用户明确说「不要保存」。好答案不应消失在聊天记录中。
   - 「A 和 B 有何不同」类问题 → `wiki/comparisons/`（`type: comparison`）
   - 跨来源综合、主题研究 → `wiki/syntheses/`（`type: synthesis`）
6. **追加 Log**：
   ```markdown
   ## [YYYY-MM-DD] query | <问题摘要>
   ```

---

### 5.3 LINT — 健康检查

**触发方式**：用户说「请 Lint」或「健康检查」

**执行频率**：每摄入 5-10 篇来源后必须运行一次；每月至少一次；重大 Query 前建议运行。

**检查项**：

| 类别 | 操作 | 权限 |
|------|------|:----:|
| **Index 一致性** | 对比 `wiki/index.md` 与实际文件：缺失条目补 `(no summary)`；指向不存在的文件标记 `[MISSING]` | ✅ 自动修复 |
| **内部链接** | 检查所有 `[[wikilink]]`：断链尝试自动修复路径；无法修复则报告用户 | ✅ 自动修复 |
| **来源引用** | 检查 Raw 字段链接是否指向存在的 raw/ 文件 | ✅ 自动修复 |
| **来源保真** | 用脚本/手动检查 wiki 中的数字、日期、引用是否能在 raw/ 中验证 | ❌ 仅报告 |
| **矛盾** | 扫描页面间冲突主张；检查 `## Contradictions` 节的格式是否正确 | ❌ 仅报告 |
| **孤儿页面** | 找出无入链的页面 | ❌ 仅报告 |
| **概念缺口** | 找出频繁提到但无独立页面的概念 | ❌ 仅报告 |
| **陈旧内容** | 标记超过 6 个月未更新且可能被新来源取代的页面 | ❌ 仅报告 |
| **质量门** | 检查页面长度（15-120 行）、是否过度堆砌、是否过于单薄 | ❌ 仅报告 |
| **Comparison slug** | 检查 `wiki/comparisons/` 中二元对比 slug 是否字母序、`-vs-` 格式是否正确 | ✅ 自动修复 |

**矛盾解决协议**：

当发现矛盾后，在相关页面的 `## Contradictions` 节中记录：

```markdown
## Contradictions
- [Source A] vs [Source B] 关于「主张 X」— status: open
  - Source A 立场：...
  - Source B 立场：...
  - 待解决：需要更多证据
```

人类决定如何解决：
- `investigating` — 收集更多来源
- `resolved` — 证据清晰，更新页面并说明理由
- `accepted-tension` — 来源确实分歧，保留双方主张并列

**Lint 后追加 Log**：
```markdown
## [YYYY-MM-DD] lint | <N> issues found, <M> auto-fixed
```

---

## 6. 命名与链接规范

### 6.1 文件命名
- **文件名**：`kebab-case.md`，全小写，空格改连字符
  - ✅ `andrej-karpathy.md`
  - ✅ `retrieval-augmented-generation.md`
  - ❌ `Andrej Karpathy.md`
  - ❌ `retrieval_augmented_generation.md`
- **Comparison 页**（`wiki/comparisons/`）：
  - 二元：`<slug-a>-vs-<slug-b>.md`，slug 按字母序排列 — 详见 [schema/comparisons-slug.md](./schema/comparisons-slug.md)
  - ✅ `claude-sonnet-4-vs-gpt-5.md`
  - ✅ `llm-wiki-vs-rag.md`
  - ❌ `gpt-5-vs-claude-sonnet-4.md`（顺序错误，应字母序）
  - ❌ `a-vs-b-vs-c.md`（多元对比用描述性 slug）

### 6.2 页面标题
- **标题**：Title Case（首字母大写）
  - ✅ `# Andrej Karpathy`
  - ✅ `# Retrieval Augmented Generation`

### 6.3 Wikilink
- 使用 `[[Page Title]]` 语法（Obsidian 兼容）
- 链接时用**页面标题**，不是文件名
  - ✅ `[[Andrej Karpathy]]`
  - ❌ `[[andrej-karpathy]]`
- **绝不留下断链** — 如果概念被提到但没有页面，创建 stub
- 每节只链接概念的**第一次出现**

### 6.4 目录深度
- `wiki/` 只支持**一级主题子目录**
- ✅ `wiki/concepts/attention-mechanism.md`
- ✅ `wiki/comparisons/rag-vs-llm-wiki.md`
- ❌ `wiki/concepts/nlp/attention-mechanism.md`

---

## 7. 质量门控

### 7.1 Bias Check（偏见检查）
每篇 concept、source、synthesis、comparison 页面必须包含：

```markdown
## Counter-arguments
[什么在反驳本页的主张？即使来源没有提到，也要主动思考。]

## Data gaps
[我们不知道什么？缺少什么？]
```

没有这两个节的页面视为不合格。

### 7.2 矛盾标注
当两个来源冲突时，使用 Obsidian callout 语法：

```markdown
> [!contradiction] Source A vs Source B
> [[source-a]] 主张 X，但 [[source-b]] 主张 Y。
> Resolution: {stated position, 或 "unresolved - flagged for review"}
```

### 7.3 验证门（Validation Gate）
- `explored: false` — AI 创建/更新时**永远**设为 false
- 只有人类审阅内容后，手动改为 `true`
- 这防止盲目信任 AI 输出

### 7.4 可信度分级
- `high` — 多来源交叉验证，领域共识
- `medium` — 单来源或有限证据，合理综合
- `low` — 推测性、早期想法、证据薄弱
- `uncertain` — 来源矛盾或信息不足

### 7.5 反堆砌 / 反单薄
- **Anti-cramming**：子主题如果有 ≥3 段内容，必须拆分为独立页面
- **Anti-thinning**：除非能写 ≥3 句有意义的内容，否则不创建新页面
- **Encyclopedia tone**：中立、基于归因的写作，不带编辑口吻

---

## 8. 特殊文件格式

### 8.1 Index（`wiki/index.md`）

```markdown
# Wiki Index

## Sources
- [[Source Title]] — 一行摘要

## Entities
- [[Entity Name]] — 一行摘要

## Concepts
- [[Concept Name]] — 一行摘要

## Syntheses
- [[Synthesis Title]] — 一行摘要

## Comparisons
- [[A vs B]] — 一行摘要
```

- 每次 Ingest/Query/Sync 后更新
- 每行 ≤120 字符
- 超过 ~50 页时，按类型分节；超过 ~100 页时，增加 `wiki/index-by-tag.md`

### 8.2 Log（`wiki/log.md`）

```markdown
# Wiki Log

## [YYYY-MM-DD] ingest | 来源标题
- Disposition: New
- Raw: raw/topic/filename.md

## [YYYY-MM-DD] query | 问题摘要

## [YYYY-MM-DD] lint | 5 issues found, 3 auto-fixed
```

- **追加-only**，绝不删除或修改历史条目
- 机器可读：`grep "^## \[" wiki/log.md | tail -10` 可获取最近 10 条操作

---

## 9. 所有权与编辑规则

| 文件/目录 | 主要所有者 | 人类编辑规则 |
|-----------|-----------|-------------|
| `raw/` | 人类 | Agent **绝对不可修改** |
| `wiki/` | Agent | 人类可直接编辑，但下次会话必须告知 Agent 改了什么 |
| `AGENTS.md` | 共同所有 | 人类和 Agent 共同迭代；变更记录在 `## Schema History` |
| `schema/` | 共同所有 | 补充 `AGENTS.md` 的细则与参考；变更同步记录 Schema History |
| `wiki/overview.md` | Agent | 人类应定期审阅 `Current Thesis`，确认或修正 |

---

## 10. 工具与集成

### 推荐工具链
| 工具 | 用途 | 优先级 |
|------|------|:------:|
| **Obsidian** | Markdown 编辑器 + Graph View | 必需 |
| **Obsidian Git** | 自动 commit/push 到 GitHub | 强烈推荐 |
| **Dataview** | 基于 frontmatter 的动态查询 | 强烈推荐 |
| **Obsidian Web Clipper** | 浏览器剪藏到 `raw/clippings/` | 强烈推荐 |
| **qmd** | 本地混合搜索（BM25 + 向量） | 300+ 页面时添加 |

### Git 同步
每次修改 `wiki/` 或 `raw/` 后：
```bash
git add . && git commit -m "wiki update: $(date +%Y-%m-%d)" && git push
```

Obsidian Git 插件可配置为每 5 分钟自动 commit。

---

## 11. Schema History

<!-- 格式：[YYYY-MM-DD] — <变更内容> | Trigger: <什么触发的>
     超过 20 条时，将旧条目归档到 .agents/schema-history.md -->

- [2026-08-22] — 初始 Schema 创建 | Trigger: 初始化 LLM Wiki 知识库
- [2026-08-22] — 新增 `wiki/comparisons/` 与 `type: comparison`；`schema/` 作为规则书补充目录；frontmatter `type` 与 wiki 子目录对齐；移除未使用的 `query` 类型；Index 增加 Comparisons 分节；补充 Comparison slug 细则（参考 Pratiyush/llm-wiki、MLMario/wiki-llm） | Trigger: Schema 修订
