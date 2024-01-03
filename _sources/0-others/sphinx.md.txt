# 使用 Sphinx 搭建知识库

## 介绍
Sphinx 可以轻松创建智能且美观的文档。

以下是 Sphinx 的一些主要功能：

- 输出格式： HTML（包括 Windows HTML 帮助）、LaTeX（用于可打印的 PDF 版本）、ePub、Texinfo、手册页、纯文本

- 广泛的交叉引用：函数、类、引文、术语表术语和类似信息的语义标记和自动链接

- 层次结构：轻松定义文档树，自动链接到兄弟姐妹、父母和孩子

- 自动索引：通用索引以及特定于语言的模块索引

- 代码处理：使用Pygments荧光笔自动突出显示

- 扩展：自动测试代码片段，通过内置扩展包含来自 Python 模块（API 文档）的文档字符串，以及通过第三方扩展提供更多功能。

- 主题：通过创建主题修改输出的外观，并重复使用许多第三方主题。

- 贡献的扩展：用户贡献的数十个扩展；其中大部分可以从 PyPI 安装。

Sphinx默认 使用reStructuredText标记语言，并且可以通过第三方扩展读取MyST markdown 。这两者都功能强大且易于使用，并且具有用于复杂文档和发布工作流程的功能。它们都基于 Docutils构建来解析和编写文档。

## 安装

```sh
pip install Sphinx
```

## 插件

### 自动构建 sphinx-autobuild 
作用：用于监听指定目录的文件变化，自动构建，“实时”生成 html。

使用方式：
```bash
# 直接在 Makefie 中添加如下命令，再使用 make livehtml 即可
livehtml:
	sphinx-autobuild "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)	
```


## 语法

### 警告
```{note} 
This is what the most basic admonitions look like.
```
支持的类型如下：
- admonition
- Attention
- caution
- danger
- error
- hint
- important
- note
- seealso
- tip
- todo
- warning

自定义
```{admonition} Another Custom Title
:class: note

Maaa! I made it look the same by setting the class.
```
