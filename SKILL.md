---
name: paper-notes
description: "Use when user asks to create reading notes for an academic paper, or says '帮我记笔记' / '写笔记' / '总结论文'. Reads full paper and generates structured notes, optionally writes to Zotero."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [paper, notes, zotero, multimodal, research, reading]
    related_skills: [arxiv, academic-paper-management]
---

# Paper Reading Notes

> 子技能，隶属于 `academic-paper-management`。本技能聚焦笔记生成流程，论文搜索、组织、BibTeX 等完整工作流见父技能。
> 模板同步位置：`academic-paper-management/templates/paper-reading-note.md`
> Zotero 笔记写入格式详见 `references/zotero-html-format.md`。

## Overview

自动生成论文阅读笔记。从 Zotero 获取论文全文，按定制化模板输出结构化笔记，可直接写入 Zotero 笔记字段。

## When to Use

- 用户说"帮我记笔记" / "写笔记" / "总结这篇论文"
- 用户提供 DOI 或论文标题要求整理笔记
- 用户要求对比多篇论文的异同

**Don't use for：** 快速问答（"这篇论文讲什么"）——直接回答即可，不需要走完整流程。

## 模板结构

```
## 基本信息
- 标题 / 作者 / 发表时间 / 会议或期刊
- DOI / arXiv 链接

## 一句话总结
用一句话说清楚这篇论文做了什么，长度不限，以看懂为准

## 问题与动机
- 解决什么问题？
- 现有方法为什么不够好？

## 整体框架（核心章节）
用结构化文本展示整体架构，让没看过论文原图的人也能理解：

格式要求：
- 不用代码块，直接在正文中排版
- 用箭头（→ ↓ ↗）表示数据流向
- 每个模块一行，缩进表示层级关系
- 模块旁用括号标注一句话作用
- 拼接、加法等操作在箭头上标注

示例：
```
Input_A → Conv1D(统一维度) → PosEnc(注入时序) ─┐
                                                ├→ CrossAttention(跨模态对齐) → FFN → SelfAttention → Output
Input_B → Conv1D(统一维度) → PosEnc(注入时序) ─┘
```

文字补充：
- 每个模块的作用和它在整体中的位置
- 关键设计选择：为什么用 A 而不是 B

## 与原始/现有框架的对比（核心章节）
- 这篇论文基于哪个经典框架改进（如 Transformer、ResNet 等）
- 改了哪些地方、没改哪些地方
- 每处改动的动机是什么
- 用对比的方式讲清楚："原始框架做 X，本文改为 Y，因为 Z"
- 如果有多篇 baseline，逐一对比关键差异

## 关键模块详解
- 核心公式（用通俗语言解释，首次出现的术语括号注释）
- 这个模块为什么有效（直觉解释）

## 实验
- 数据集和评估指标
- 主要结果（关键数字）
- 消融实验发现（哪个模块贡献最大）

## 局限性
- 论文自身提到的
- 我观察到的

## 对我的研究的启发
- 与 VLM / 多模态融合 / 跨模态对齐的关系
- 可借鉴的点
- 可改进的方向
- 社区讨论中的有价值观点（来自 Grok Search）

## 延伸阅读
从本论文的参考文献中，挑 2-3 篇最值得读的，每篇附上：
- 论文标题和链接
- 推荐理由（为什么值得读、跟这篇论文的关系是什么）
```

## 工作流程

### Step 1：获取论文全文 + 网络资料
**1a. 从 Zotero 获取论文全文：**
```
mcp_zotero_zotero_get_item_fulltext(item_key=...)
```
如果 fulltext 失败，退回到 metadata + abstract：
```
mcp_zotero_zotero_get_item_metadata(item_key=..., include_abstract=True)
```

**1b. 用 Grok Search 搜索网络资料：**
搜索论文标题、方法名、GitHub issue、博客讲解等，补充论文本身没有的视角：
```
mcp_grok_search_web_search(query="论文标题 方法讲解 OR tutorial OR explained")
mcp_grok_search_web_search(query="论文标题 GitHub issues OR discussion")
```
重点关注：
- 博客/知乎/Reddit 上的通俗讲解
- 后续工作的改进或批评
- 代码实现中的注意事项
- 社区讨论中的常见问题

### Step 2：按模板生成笔记
- 用中文输出
- 技术术语首次出现时括号注释英文
- 实验结果保留关键数字
- 评价要客观，指出优缺点
- 结合网络资料补充：论文没讲清楚的地方、社区的常见疑问、后续工作的评价

### Step 3：写入 Zotero（需写入权限）

**写入方式（HTML 直接写入）：**

Zotero 笔记 API 直接写入 HTML，笔记到达客户端时已是渲染好的格式。

**写入格式规则：**
- 整体包裹：`<div data-schema-version="9">...</div>`
- 行内公式：`<span class="math">$...$</span>`
- 块级公式：单独一个 `<p><span class="math">$...$</span></p>`
- 所有公式都用单 `$`，不用 `$$`
- LaTeX 内容原样写入，不转义
- 标题由 `note_title` 参数处理，内容里<strong>不要写 `<h1>`</strong>，直接从 `<h2>` 开始
- 二级标题用 `<h2>`
- 粗体用 `<strong>`
- 列表用 `<ul>/<li>` 或 `<ol>/<li>`
- 链接用 `<a href="url">text</a>`
- 段落用 `<p>`
- 不要用 `[MD_START]`/`[MD_END]` 标记

示例格式：
```html
<div data-schema-version="9">
<h2>章节</h2>
<p>正文，<strong>加粗</strong>，<span class="math">$E=mc^2$</span></p>
<p><span class="math">$Y = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$</span></p>
<ul>
<li>列表项</li>
</ul>
<a href="https://arxiv.org/abs/xxxx">链接文字</a>
</div>
```

**⚠️ 重要：** `zotero_update_note` 不会触发插件转换。如需修改已写入的笔记，必须先删除再重建（`zotero_delete_note` + `zotero_create_note`）。

### Step 4：询问是否导入新论文
如果用户提到的论文不在 Zotero 中：
```
mcp_zotero_zotero_add_by_url(url=..., tags=[...])
```

## 自定义选项

用户可要求调整模板：
- 精简版：只保留"一句话总结 + 方法 + 实验 + 局限性"
- 深度版：增加"公式推导详解" / "代码实现要点"
- 对比版：同时对比 2-3 篇论文

## Common Pitfalls

1. **论文太长** — fulltext 超过 12K tokens 时会被截断，优先从摘要和关键章节提取，不要猜测被截断的内容。
2. **Zotero 写入 403** — API Key 没有写入权限，笔记直接输出给用户即可，不要反复重试。
3. **论文类型不同** — Survey 类论文不需要"实验"部分，改为"分类体系 + 关键洞察"。
4. **笔记语言** — 默认中文，除非用户要求英文。
5. **术语翻译** — 保留英文原文（如 Crossmodal Attention），不要强行翻译。
6. **Zotero 笔记标题** — Zotero 用笔记第一行作为显示标题。`note_title` 参数会创建标题，内容里不要再写 `<h1>`，否则会重复。直接从 `<h2>` 开始写内容。
9. **Zotero 9 插件兼容性** — manifest 格式才是关键，不是签名。正确格式见 `references/zotero7-plugin-dev.md`。

## Verification Checklist

- [ ] 获取到论文全文或摘要
- [ ] 模板所有必要章节已填写
- [ ] 实验数据准确（有数字支撑）
- [ ] 评价客观（既有优点也有局限）
- [ ] Zotero 写入成功（或笔记已输出给用户）
