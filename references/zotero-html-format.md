# Zotero 笔记 HTML 写入格式

## 整体结构

用 `<div data-schema-version="9">` 包裹所有内容：

```html
<div data-schema-version="9">
<h1>论文笔记标题</h1>
<h2>章节</h2>
<p>正文内容</p>
</div>
```

## 公式格式

所有公式统一用单 `$`，不用 `$$`：

- **行内公式**：`<span class="math">$E=mc^2$</span>`
- **块级公式**：单独一个 `<p><span class="math">$公式$</span></p>`

LaTeX 内容原样写入，不转义。

## HTML 标签对照

| Markdown | HTML |
|----------|------|
| `# 标题` | `<h1>标题</h1>` |
| `## 标题` | `<h2>标题</h2>` |
| `**加粗**` | `<strong>加粗</strong>` |
| `*斜体*` | `<em>斜体</em>` |
| `- 列表` | `<ul><li>列表</li></ul>` |
| `1. 列表` | `<ol><li>列表</li></ol>` |
| `[文字](url)` | `<a href="url">文字</a>` |
| `段落` | `<p>段落</p>` |

## Zotero API 行为

1. `zotero_create_note`：直接写入 HTML，笔记到达客户端时已是渲染好的格式
2. `zotero_update_note`：**不触发**插件转换，内容会显示为原始文本
3. 修改笔记必须：`zotero_delete_note` + `zotero_create_note`

## 标题显示

- Zotero 用笔记**第一行**作为显示标题
- `note_title` 参数虽不生效，但 API 要求必填（缺失报 validation error）
- 所以必须传 `note_title`（随便填），同时内容第一行写 `<h1>标题</h1>`

## 完整示例

```html
<div data-schema-version="9">
<h1>论文笔记标题</h1>
<h2>基本信息</h2>
<ul>
<li><strong>标题</strong>：Paper Title</li>
<li><strong>作者</strong>：Author Names</li>
<li><strong>链接</strong>：<a href="https://arxiv.org/abs/xxxx">arXiv</a></li>
</ul>
<h2>整体框架</h2>
<p>输入 <span class="math">$X \in \mathbb{R}^{T \times d}$</span>：</p>
<p><span class="math">$Y = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$</span></p>
<h2>实验</h2>
<p>主要结果：<strong>Acc=81.6%</strong></p>
</div>
```
