# Paper Notes Skill for Hermes Agent

自动为学术论文生成结构化阅读笔记，支持直接写入 Zotero 并渲染 LaTeX 公式。

## 功能

- 从 Zotero 获取论文全文，按定制模板生成中文笔记
- 支持 LaTeX 公式（行内 `$...$` 和块级）
- 直接写入 Zotero 笔记，到达客户端时已是渲染好的格式
- 适用于多模态、NLP、CV 等领域的论文

## 安装

将 `paper-notes` 目录放到 Hermes Agent 的 skills 目录下：

```bash
cp -r paper-notes ~/.hermes/skills/research/
```

## 使用方法

在 Hermes Agent 中说：

- "帮我记笔记" / "写笔记" / "总结这篇论文"
- 提供 DOI 或论文标题要求整理笔记
- 要求对比多篇论文的异同

## 笔记模板

生成的笔记包含以下章节：

1. **基本信息** — 标题、作者、发表时间、链接
2. **一句话总结** — 用一句话说清楚论文做了什么
3. **问题与动机** — 解决什么问题，现有方法为什么不够好
4. **整体框架** — 用结构化文本展示架构
5. **与原始框架的对比** — 改了哪些地方，为什么
6. **关键模块详解** — 核心公式和直觉解释
7. **实验** — 数据集、主要结果、消融实验
8. **局限性** — 论文自述和观察到的问题
9. **对我的研究的启发** — 可借鉴的点和可改进的方向
10. **延伸阅读** — 2-3 篇推荐的相关论文

## Zotero 写入格式

笔记通过 HTML 直接写入 Zotero，格式规则：

- 整体包裹：`<div data-schema-version="9">...</div>`
- 行内公式：`<span class="math">$...$</span>`
- 块级公式：单独一个 `<p><span class="math">$...$</span></p>`
- 所有公式用单 `$`，不用 `$$`
- LaTeX 内容原样写入，不转义

详见 [references/zotero-html-format.md](references/zotero-html-format.md)

## 注意事项

- 论文全文超过 12K tokens 时会被截断，优先从摘要和关键章节提取
- Survey 类论文改用"分类体系 + 关键洞察"，不需要"实验"部分
- 默认中文输出，术语保留英文原文
- 修改已写入的笔记必须删除重建（`zotero_delete_note` + `zotero_create_note`）

## 文件结构

```
paper-notes/
├── SKILL.md                              # 技能定义文件
├── README.md                             # 本文件
└── references/
    ├── zotero-html-format.md             # Zotero HTML 写入格式说明
    └── zotero7-plugin-dev.md             # Zotero 7+ 插件开发参考
```

## 依赖

- Hermes Agent
- Zotero MCP 工具（用于读写 Zotero 笔记）
- Zotero 9.0.3+（Windows/macOS/Linux）
- Better Notes 插件（可选，用于 LaTeX 渲染）

## 许可证

MIT License
