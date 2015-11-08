---
layout: blog
categories: Linux
title: Vim 多文件编辑：窗口
tags: Vim Bash window 
---

**标签页(tab)**、**窗口(window)**、**缓冲区(buffer)**是Vim多文件编辑的三种方式，它们可以单独使用，也可以同时使用。
它们的关系是这样的：

> A buffer is the in-memory text of a file.  A window is a viewport on a buffer.  A tab page is a collection of windows.
> --[vimdoc][vim-window-doc]

本文主要介绍Vim窗口的创建与维护，另外两种编辑方式的使用可以参考： 
[Vim 多文件编辑：缓冲区][vim-buffer]和 [Vim 多文件编辑：标签页][vim-tabpage]。先上图：

![vim window][vim-window]

<!--more-->

# 打开关闭

使用`-O`参数可以让Vim以分屏的方式打开多个文件：

```bash
vim -O main.cpp my-oj-toolkit.h
```

> 使用小写的`-o`可以水平分屏。

## 打开关闭命令

在进入Vim后，可以使用这些命令来打开/关闭窗口：

```vim
:sp[lit] {file}     水平分屏
:sv[iew] {file}     水平分屏，以只读方式打开
:vs[plit] {file}    垂直分屏
:clo[se]            关闭当前窗口
```

> 上述命令中，如未指定file则打开当前文件。

## 打开关闭快捷键

上述命令都有相应的快捷键，它们有共同的前缀：`Ctrl+w`。

```
Ctrl+w s        水平分割当前窗口
Ctrl+w v        垂直分割当前窗口
Ctrl+w q        关闭当前窗口
```

# 切换窗口

切换窗口的快捷键就是`Ctrl+w`前缀 + `hjkl`：

```
Ctrl+W h        切换到左边窗口
Ctrl+W j        切换到下边窗口
Ctrl+W k        切换到上边窗口
Ctrl+W l        切换到右边窗口
Ctrl+W w        遍历切换窗口
```

> 还有`t`切换到最上方的窗口，`b`切换到最下方的窗口。

# 移动窗口

分屏后还可以把当前窗口向任何方向移动，只需要将上述快捷键中的`hjkl`大写：

```
Ctrl+W H        向左移动当前窗口
Ctrl+W J        向下移动当前窗口
Ctrl+W K        向上移动当前窗口
Ctrl+W L        向右移动当前窗口
```

# 调整大小

调整窗口大小的快捷键仍然有`Ctrl+W`前缀：

```
Ctrl+W +        增加窗口高度
Ctrl+W -        减小窗口高度
Ctrl+W =        统一窗口高度
```

[tmux]: {% post_url 2015-11-06-tmux-startup %}
[tree]: {% post_url 2015-11-04-vim-ide %}
[vim-buffer]: /2015/11/17/vim-buffer.html
[vim-tabpage]: /2015/11/12/vim-tabpage.html
[vim-window]: /assets/img/blog/vim-window@2x.png
[vim-window-doc]: http://vimdoc.sourceforge.net/htmldoc/windows.html
