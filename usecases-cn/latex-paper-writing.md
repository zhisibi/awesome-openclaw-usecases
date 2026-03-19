搭一套本地 LaTeX 环境，很多人都经历过：

- 装 TeX Live / MacTeX，占几 GB 空间
- 到处是 package 冲突和编译 Warning
- 在编辑器和 PDF 查看器之间来回切换，调版面很痛苦

这个用例的目标是：

> 不在本机装 LaTeX，把 **OpenClaw + Prismer 提供的 LaTeX 服务**当远程编译器，让 Agent 和你一起写论文、改结构、查错误、导出 PDF。

---

## 一、整体思路：LaTeX 变成一个「远程编译服务」

原文中的工作流是这样设计的：

- 你通过对话描述想要写的论文结构、内容
- Agent 使用 `latex-compiler` skill：
  - 选择模版（article / IEEE / beamer / 中文模版等）
  - 填写 / 修改 LaTeX 源码
  - 调用远端的 LaTeX 服务编译
  - 把生成好的 PDF 直接返回给你预览

你只需要：

- 在聊天里说“把这段改写成更正式的 related work 风格”
- 或者说“这里多加一个表格，展示三个模型的比较”

其它繁琐的事情（环境、编译、错误日志）都交给 Agent 处理。

---

## 二、依赖组件：Prismer + latex-compiler skill

原用例基于 [Prismer](https://github.com/Prismer-AI/Prismer) 提供的 LaTeX 服务：

- 通过 Docker 一键起一个容器，里面装好了完整的 TeX Live
- 对外暴露一个 LaTeX 编译 HTTP API（默认 8080 端口）

核心技能是 `latex-compiler`：

- `latex_compile`
- `latex_preview`
- `latex_templates`
- `latex_get_template`

这些工具负责：

- 获取模版
- 传入 LaTeX 源码并编译
- 返回编译产物（PDF）或错误日志

---

## 三、部署步骤（简版）

1. Clone 并启动 Prismer：

```bash
git clone https://github.com/Prismer-AI/Prismer.git && cd Prismer
docker compose -f docker/docker-compose.dev.yml up
```

2. 确保 LaTeX 服务器在 8080 端口正常运行
3. 在 OpenClaw 网关中启用 `latex-compiler` skill

之后，你就可以用类似下面的 Prompt：

```text
Help me write a research paper in LaTeX. Here's my workflow:

1. Start from the IEEE template (or article/beamer depending on what I need)
2. When I describe a section, generate the LaTeX source for it
3. After each major edit, compile and preview the PDF so I can check formatting
4. If there are compilation errors, read the log and fix them automatically
5. When I provide BibTeX entries, add them to the bibliography and recompile

Use xelatex if I need Chinese/CJK support, otherwise default to pdflatex.
Always run 2 passes for cross-references.
```

---

## 四、典型使用场景

1. **写新论文**：
   - “帮我用 IEEE 模版起一个题为『A Survey of LLM Agents』的论文框架。”
   - Agent：
     - 拉取 IEEE 模版
     - 生成标题、作者、摘要、引言等基本结构
     - 编译并返回第一版 PDF

2. **改写 / 重构章节**：
   - “把 related work 这一节改写成更学术、引用更规范的版本。”
   - Agent：修改 LaTeX 源码，重新编译

3. **处理编译错误**：
   - LaTeX 报 miss $ or extra alignment tab 的时候，让 Agent 看 log 并自动修复

4. **管理 BibTeX**：
   - 你把 `.bib` 内容丢给 Agent，让它：
     - 追加到文献数据库
     - 在文中插入 `\cite{}`
     - 重新运行 `bibtex + latex` 编译流程

---

## 五、中文 / CJK 支持

原文里也强调了一点：

- 当需要中文 / CJK 支持时，优先使用 `xelatex`
- 并在模版中预置好相应的字体配置

你可以直接对 Agent 说：

> “这篇论文包含中文，请使用 xelatex，并选一个合适的中文字体。”

Agent 可以：

- 在导言区加上：
  - `\usepackage{xeCJK}`
  - 字体设置
- 之后所有中文内容就都能正常编译

---

## 六、这种写 LaTeX 的方式有什么优势？

1. **本机零环境负担**：
   - 不装 TeX Live，不折腾包管理
   - 只要能连上网关，就能写和编译

2. **写作流程更「对话化」**：
   - 不用每次自己改源文件再按快捷键编译
   - 把修改想法告诉 Agent，让它来写 LaTeX 代码

3. **更适合团队协作**：
   - 多个人都通过同一个 Agent / 网关协作编辑
   - LaTeX 源文件可以放在 Git 仓库里统一管理

---

## 原文链接

- [LaTeX Paper Writing](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/latex-paper-writing.md)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/latex-paper-writing.md
---
