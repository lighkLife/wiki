# 使用 Sphinx 搭建知识库

## 介绍

Sphinx 可以轻松创建智能且美观的文档。书写方式：

- 默认使用 [reStructuredText](https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html#top) 标记语言
- 可以通过第三方扩展读取 [MyST markdown](https://myst-parser.readthedocs.io/en/latest/syntax/typography.html)

以下是 Sphinx 的一些主要功能：

- 输出格式： HTML（包括 Windows HTML 帮助）、LaTeX（用于可打印的 PDF 版本）、ePub、Texinfo、手册页、纯文本

- 广泛的交叉引用：函数、类、引文、术语表术语和类似信息的语义标记和自动链接

- 层次结构：轻松定义文档树，自动链接到兄弟姐妹、父母和孩子

- 自动索引：通用索引以及特定于语言的模块索引

- 代码处理：使用 Pygments 荧光笔自动突出显示

- 扩展：自动测试代码片段，通过内置扩展包含来自 Python 模块（API 文档）的文档字符串，以及通过第三方扩展提供更多功能。

- 主题：通过创建主题修改输出的外观，并重复使用许多第三方主题。

- 贡献的扩展：用户贡献的数十个扩展；其中大部分可以从 PyPI 安装。



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

## 提示块

例子

```
```{note} 
This is **{note}** look like.
```

```{note}
This is **{note}** look like.
```

```{admonition} admonition
This is **{admonition}** look like.
```

```{Attention}
This is **{Attention}** look like.
```

```{caution}
This is **caution** look like.
```

```{danger}
This is **danger** look like.
```

```{error}
This is **error** look like.
```

```{hint}
This is **hint** look like.
```

```{important}
This is **important** look like.
```

```{seealso}
This is **seealso** look like.
```

```{tip}
This is **tip** look like.
```

```{todo}
This is **todo** look like.
```

```{warning}
This is **warning** look like.
```

自定义

```
```{admonition} 自定义
:class: note

Maaa! I made it look the same by setting the class.
```

## 代码块

### 高亮展示

在`config.py`添加 `myst` 扩展, 并启用 `attrs_block` 功能

```python
extensions = ['myst_parser']

myst_enable_extensions = [
        "attrs_block",
]
```

例子

```
{lineno-start=1 emphasize-lines="2,3"}
```python
a = 1
b = 2
c = 3
```

{lineno-start=1 emphasize-lines="2,3"}

```python
a = 1
b = 2
c = 3
```
