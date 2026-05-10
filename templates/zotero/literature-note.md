---
type: literature-note
citekey: {{citekey}}
title: "{{title | replace('"', '\\"')}}"
aliases: ["@{{citekey}}"]
authors: "{{authors}}"
firstAuthor: "{% for c in creators %}{% if c.creatorType == 'author' and loop.first %}{{c.lastName}}{% endif %}{% endfor %}"
year: {{date | format("YYYY")}}
itemType: {{itemType}}
{%- if publicationTitle %}
venue: "{{publicationTitle | replace('"', '\\"')}}"
{%- endif %}
{%- if DOI %}
doi: "{{DOI}}"
{%- endif %}
{%- if url %}
url: "{{url}}"
{%- endif %}
zotero: "{{desktopURI}}"
tags: [literature{%- for t in tags %}, {{t.tag | replace(" ", "-") | lower}}{%- endfor %}]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: {{importDate | format("YYYY-MM-DD")}}
updated: {{importDate | format("YYYY-MM-DD")}}
ingested_to_wiki: false # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page:              # e.g. "[[wiki/sources/{{citekey}}]]"
---

# {{title}}

> [!info] @{{citekey}} · {% for c in creators %}{% if c.creatorType == "author" and loop.first %}{{c.lastName}}{% if creators.length > 1 %} et al.{% endif %}{% endif %}{% endfor %} · {{date | format("YYYY")}}{% if publicationTitle %} · *{{publicationTitle}}*{% endif %}
> [Open in Zotero]({{desktopURI}}){% if DOI %} · [DOI](https://doi.org/{{DOI}}){% endif %}{% if url %} · [URL]({{url}}){% endif %}{% for att in attachments %}{% if att.path and att.path.endsWith("pdf") %} · [PDF](file://{{att.path | replace(" ", "%20")}}){% endif %}{% endfor %}

## Abstract

> [!abstract]- Click to expand
> {{abstractNote | replace("\n", "\n> ")}}

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

-

## 高亮颜色约定（个人 convention）

> 🟡 **Yellow** = 关键论点 / takeaway
> 🔴 **Red** = 我有异议 / 可疑结论 / 论文改进点
> 🟢 **Green** = 可借鉴的方法 / 公式 / trick
> 🔵 **Blue** = 后续要追溯的引用
> 🟣 **Purple** = 与我 thesis 直接相关
> ⚫ **Gray** = 背景 / 术语定义

## Annotations

{% persist "annotations" %}
{%- set newAnnots = annotations | filterby("date", "dateafter", lastImportDate) -%}
{%- if newAnnots.length > 0 %}

### Imported {{importDate | format("YYYY-MM-DD HH:mm")}}
{% for a in newAnnots %}
{%- set badge = "⚪" -%}
{%- if a.colorCategory == "Yellow" %}{% set badge = "🟡" %}{% endif -%}
{%- if a.colorCategory == "Red" %}{% set badge = "🔴" %}{% endif -%}
{%- if a.colorCategory == "Green" %}{% set badge = "🟢" %}{% endif -%}
{%- if a.colorCategory == "Blue" %}{% set badge = "🔵" %}{% endif -%}
{%- if a.colorCategory == "Purple" %}{% set badge = "🟣" %}{% endif -%}
{%- if a.colorCategory == "Gray" %}{% set badge = "⚫" %}{% endif -%}
{%- if a.annotatedText %}
- {{badge}} **p.{{a.pageLabel or a.page}}** — {{a.annotatedText | replace("\n", " ")}} [⤴]({{a.desktopURI}})
{%- endif %}
{%- if a.imageRelativePath %}
  - ![[{{a.imageRelativePath}}]]
{%- endif %}
{%- if a.comment %}
  - 💬 *我的批注*：{{a.comment | replace("\n", "  \n      ")}}
{%- endif %}
{% endfor %}
{%- endif %}
{% endpersist %}

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

1.
2.
3.

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

-

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

- [ ]

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/{{citekey}}.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/{{citekey}}.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。
