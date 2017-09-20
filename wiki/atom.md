---
layout: docs
title:  "Atom"
categories: Atom
toc: true
permalink: /wiki/atom
---

[flight-manual](http://flight-manual.atom.io/)

[资料](https://github.com/futantan/atom)

## Snippets

1. 使用
例如在编辑文件new.html是，输入`html`，然后键入<kbd class="platform-all">Tab</kbd>。

2. 自定义
snippets.cson:
```cson
'.source.js':
  'console.log':
    'prefix': 'log'
    'body': 'console.log(${1:"crash"});$2'
```
多行Snippets
```cson
'.source.js':
  'if, else if, else':
    'prefix': 'ieie'
    'body': """
      if (${1:true}) {
        $2
      } else if (${3:false}) {
        $4
      } else {
        $5
      }
    """
```

## Git

### 恢复

当你修改了某个文件,然后发现改得不满意,希望恢复文件到最后一次提交的状态,可以使用`Cmd+Alt+Z`或者`Checkout Head Revision`, 此命令将会放弃你对文件所有的修改, 直接将文件恢复为最后一次提交的版本
相当于Git命令`git checkout HEAD -- filename`和`git reset HEAD -- filename`。

如果恢复文件后发现还是改过以后的好,可以使用`Cmd+Z`来撤销刚才的修改。

### 显示状态

在前文中讲过,我们可以通过`Cmd+T`/`Cmd+P`列出所有项目中的文件, 或`Cmd+B`列出所有当前打开的文件,
或是`Cmd+Shift+B`来列出所有新建的或更改过的文件。所有的这些方法都会在弹出的文件列表的右边以图标的形式显示文件的状态，特别是`Cmd+Shift+B`, 它会列出所有未跟踪或是更改过的文件,相当于`Toggle Git Status Finder`命令。

### 编辑提交信息

你可以通过如下命令将Atom设置为Git的默认编辑器

git config --global core.editor "atom --wait"
1
1
这样当你提交时就会使用Atom来编辑提交信息
并且Atom还支持提交信息的高亮

### GitHub支持

如果你的代码托管在GitHub上,掌握下列命令可以让你更方便地工作

* `Alt+G` `O` 在GitHub上打开当前文件
* `Alt+G` `B` 在GitHub上用Blame方式打开当前文件
* `Alt+G` `H` 在GitHub上用History方式打开当前文件
* `Alt+G` `C` 将当前文件在GitHub上的URL复制到剪切板
* `Alt+G` `R` 在GitHub上比较分支

## Plugins

### atom-beautify

[atom-beautify](https://atom.io/packages/atom-beautify)

缺省快捷键：`cltr-alt-B`

```json
'.editor':
  'ctrl-alt-b': 'atom-beautify:beautify-editor'
```

### atom-typescript

[atom-typescript](https://atom.io/packages/atom-typescript)
