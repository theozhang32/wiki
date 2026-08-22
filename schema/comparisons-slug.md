# Comparison 页 Slug 命名细则

> 本文件是 `AGENTS.md` §6.1 的补充。Agent 创建 `wiki/comparisons/` 页面时遵循此处规则。

## 参考来源

| 项目 | 惯例 | 链接 |
|------|------|------|
| **Pratiyush/llm-wiki** | 二元对比 slug：`<slug-a>-vs-<slug-b>`；**按字母序**排列两侧 slug，保证每对实体只有一个规范 URL（如 `ClaudeSonnet4-vs-GPT5`，永不 `GPT5-vs-ClaudeSonnet4`） | [compare.py #58](https://github.com/Pratiyush/llm-wiki/commit/2a31a49) |
| **MLMario/wiki-llm** | Comparison 页放 `wiki/comparisons/`，用于 A vs B 并排分析；**谨慎创建**，仅在直接对比有价值时使用 | [wiki-llm README](https://github.com/MLMario/wiki-llm) |
| **GitHub Docs** | 文件名 = `title` frontmatter 的 kebab-case；标点可省略 | [content/README.md](https://github.com/github/docs/blob/main/content/README.md) |

## Slug 模板

### 1. 二元对比（首选）

```
wiki/comparisons/<slug-a>-vs-<slug-b>.md
```

| 字段 | 规则 |
|------|------|
| **slug-a / slug-b** | 对比主体的 kebab-case slug，通常取自对应 [[Entity]] 或 [[Concept]] 页文件名（去掉 `.md`） |
| **顺序** | **字母序**：`slug-a` 必须 ≤ `slug-b`（ASCII/Unicode 字典序）。写反则 Lint 应报告并自动修正 |
| **连接符** | 固定为 `-vs-`（小写），不用 `versus`、`compare`、`对比` |
| **title** | Title Case：`"Claude Sonnet 4 vs GPT-5"`（`vs` 小写） |

**示例**

| 主体 | 文件名 | title |
|------|--------|-------|
| RAG vs LLM Wiki | `llm-wiki-vs-rag.md` | `LLM Wiki vs RAG` |
| Claude Sonnet 4 vs GPT-5 | `claude-sonnet-4-vs-gpt-5.md` | `Claude Sonnet 4 vs GPT-5` |
| PostgreSQL vs SQLite | `postgresql-vs-sqlite.md` | `PostgreSQL vs SQLite` |

### 2. 多元对比（3+ 主体）

不用 `-vs-` 链式拼接（避免 `a-vs-b-vs-c` 歧义）。改用**描述性** kebab-case：

```
wiki/comparisons/vector-database-landscape.md
wiki/comparisons/agent-memory-approaches.md
```

title 示例：`Vector Database Landscape`、`Agent Memory Approaches`。

### 3. 与 Wikilink 的关系

- 页面内用 `[[Entity Name]]` 链接对比主体
- 文件名 slug **不必**与 title 逐字对应（二元对比以 `-vs-` 规则为准）
- 创建 Comparison 页后，在相关 Entity/Concept 页的 `## Related Pages` 或 `## Relations` 中回链

## 创建阈值

- **2 个可命名主体** + 用户明确要问「A 和 B 有何不同」→ 创建 Comparison 页
- 对比结论可并入某 Concept 页的一段 → **不**单独建 Comparison（避免页面碎片化）
- 同一对主体只保留**一个**规范 slug；新来源更新该页，不新建 `b-vs-a`

## Slug 生成伪代码

```python
def comparison_slug(slug_a: str, slug_b: str) -> str:
    a, b = sorted([slug_a, slug_b])
    return f"{a}-vs-{b}"
```
