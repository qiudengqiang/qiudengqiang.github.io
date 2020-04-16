---
title: VSCode 使用技巧(一)
date: 2019-05-04 10:01:47
tags: [VSCode]
categories: VSCode
---

# VS Code Note 01

## 命令面板
`Cmd+Shift+P / F1 `：打开命令面板，是VS Code的主要交互界面

## 界面概览
文件资源管理器：`Ctrl+Shift+E`
跨文件搜索：`Ctrl+Shift+F`
源代码管理：`Ctrl+Shift+G`
启动和调试：`Ctrl+Shift+D`
管理扩展：`Ctrl+Shift+X`
查看错误和警告：`Ctrl+Shift+M`
![vs-view-preview](/images/tech/vs-view-preview.png)

## 命令行的使用
如果你是在 macOS 上使用，安装后打开命令面板，搜索`shell` 命令：在 PATH 中安装 `code` 命令并执行，然后重启终端模拟就可以了。
![vs-shell-on-terminal](/images/tech/vs-shell-on-terminal.png)

## OPTIONS（常用命令）
```bash
  -n --new-window                   Force to open a new window..
  -r --reuse-window  Force to open a file or folder in an already opened window.
  -g --goto <file:line[:character]> Open a file at the path on the specified line and character.
  -d --diff <file> <file>           Compare two files with each other.
```
### 通过命令行打开文件，并滚动至特定的`行`
比如输入`code -r -g package.json:128`命令，你就可以打开 package.json 这个文件，然后自动跳转到 128 行。适用于某文件错误的行数的快速定位

### 比较两个文件的内容
只需使用 `-d`参数，并传入两个文件路径，比如输入 `code -r -d a.txt b.txt`命令

### 打开管道内容
将命令行 ls 的执行结果在 VS Code 的编辑器中打开 `ls | code -`
注：[管道的说明](https://zh.wikipedia.org/zh-hans/%E7%AE%A1%E9%81%93_(Unix)

### 如何在终端和文件之间快速切换
快捷键：Ctrl + `

# VS Code Note 02

## 光标的快捷键移动操作
1. 首先是针对单词的光标移动: `Option + 左右方向键` 同时按住 Option 和方向键，那么光标移动的颗粒度就变成了单词，你就可以在文档中以单词为单位不停地移动光标了。
2. 第二种方式是把光标移动到行首或者行末：`Cmd + 左右方向键`
3. 接下来一种是对于代码块的光标移动：`Cmd + Shift + \`，就可以在这对花括号之间跳转。
4. 最后是移动到文档的第一行或者最后一行：`Cmd + 上下方向键`

## 文本选择
基于以上快捷键，只需要多按一个 Shift 键，就可以在移动光标的同时选中其中的文本。

## 删除操作
1. 删除光标右侧所有内容：`Cmd + fn + delete`
2. 删除光标左侧所有内容：`Cmd + delete `
3. 删除单词内右侧的字符：`Option + fn + delete`
4. 删除单词内左侧的字符：`Option + delete` 
 
## 自定义快捷键
首先你可以打开命令面板（`Cmd + Shift + p`），搜索“打开键盘快捷方式”然后执行，这时你将看到相对应的界面。
![vs-keyboard-shortcuts](/images/tech/vs-keyboard-shortcuts.png)

比如，你可以搜索“选择括号内所有内容”，双击，按下`Cmd + Shift + ]`，然后按下回车，这个快捷键就绑定上了。
比如，你通过搜索 “cmd+backspace”这组快捷键，发现它对应的命令是“删除左侧所有内容”，但你不希望使用这个命令，那你就可以通过右键选择删除该快捷键的绑定。


# VS Code Note 03

## 代码行编辑
1. 删除当前行代码： `Cmd + Shift + K `
2. 剪切当前行代码： `Cmd + x`
3. 在当前代码上面开始一行时：`Cmd + Shift + Enter`
4. 在当前代码下面开始一行时：`Cmd + Enter`
5. 移动当前选中行代码：`Option + 上下方向键`
6. 赋值当前选中行代码：`Option + Shift + 上下方向键`

## 编程语言相关的命令
1. 添加注释：`Cmd + /`
2. 代码格式化：`Option + Shift + F`，VS Code 也会根据你当前的语言，选择相关的插件。当然，前提条件是你已经安装了相关插件。
3. 选中的代码格式化：`Cmd + k` or ` Cmd + F`，这样只有这段被选中的代码才会被格式化。

## 其他小技巧
1. 调换字符的位置：`Ctrl + t`
2. 是调整字符的大小写：你可以选中一串字符，然后在命令面板里运行“转换为大写(transform to uppercase)”或 “转换为小写”, 来变换字符的大小写。
3. 是合并代码行：` Ctrl + j`，可以把较短的代码合并至一行，而不需要不断地调整光标、删除换行符。
4. 是撤销光标的移动和选择：`Cmd + U`


# VS Code Note 04

## 创建多个光标
### 两个特别命令
1. `Cmd + D` 适用情况比较特别：处理多次出现的“相同”单词。如果要处理的文本并不是相同的，那么这个方法就不适用了。
2. `Option + Shift + i` 首先选择多行代码，然后按下 “Option + Shift + i” ,这样操作的结果是：每一行的最后都会创建一个新的光标。

## 快捷键跳转
### 多文件跳转
1. `Ctrl + Tab` 在正在操作的文件列表中切换文件
2. `Cmd + P` 打开最近的文件列表，并支持搜索功能
「小技巧」 `Cmd + Enter` 当找到文件后按下这个快捷键可在新窗口中打开该文件

### 行跳转
`Ctrl + G` 跳转到指定的代码行 
「高级组合技巧」：按下`Cmd + P` 直接输入`文件名：行号` eg:`main.css:3`

### 符号跳转
`Cmd + R` 能够看到当前文件里的所有符号(包括类，方法等)。

### 定义 (Definition) 和实现 (implementation) 跳转
`Cmd + F12` 选中引用跳转到函数的实现位置

### 引用 (Reference) 跳转
`Shift + F12` 选中函数或类按下这个快捷键就可以出现一个引用的列表，选择后内嵌的编辑器里便展示相应的引用代码

# VS Code Note 04
## 玩转鼠标
### 选择文本
1. 在VS Code中，你单击鼠标左键就可以把光标移动到相应的位置。
2. 而双击鼠标左键，则会将当前光标下的单词选中。
3. 连续三次按下鼠标左键，则会选中当前这一行代码。
4. 最后是连续四次按下鼠标左键，则会选中整个文档。

### 文本编辑
在 VS Code中，我们除了能够使用鼠标来选择文本以外，还能够使用鼠标对文本进行一定程度的修改，我们把它称为拖放功能（drag and drop）。
比如在示例代码中，我们选中 bar 这个函数，然后将鼠标移到这段选中的代码之上，按下鼠标左键不松开。这时你可以看到，鼠标指针已经从一条竖线，变成了一个箭头。这时候我们移动鼠标的话，就可以把这段文本拖拽到我们想要的位置。
![](/images/tech/vs-bar-drag-and-drop.gif)

### 悬停提示窗口
在 VS Code 的编辑器里使用鼠标的过程中，当你的鼠标移动到某些文本上之后，稍待片刻就能看到一个悬停提示窗口。这个窗口里会显示跟鼠标下文本相关的信息。

- 方法(Method)

比如，在示例代码中，当我把鼠标移动到第五行 foo 上后，悬停提示窗口里展示了 foo的类型信息，它告诉我们 foo是一个函数，不需要任何的参数，返回值是 void。
![](/images/tech/vs-method-hover-window.gif)

如果我把鼠标移动到 foo 上面时，按下 Cmd 键，则能够在悬停提示窗口里直接看到 foo的实现。
![](/images/tech/vs-method-hover-window-detail.gif)

- 变量(Field)

在 JavaScript 或者 Java 这样的编程语言中，当我们把鼠标移动到某个变量上时，我们能够看到这个变量的定义信息。而在 CSS 中，当我们把鼠标移动到一个 CSS 规则上时，我们能看到的则是一段能够让这个 CSS 规则生效的 HTML 的样例代码。
![](/images/tech/vs-field-hover-window.gif)

### 代码跳转和链接
我们还是把鼠标移动到示例代码的第五行 foo 上，然后按下 Cmd 键，这时候 foo下面出现了一个下划线。然后当我们按下鼠标左键，就跳转到了 foo函数的定义处。
![](/images/tech/vs-code-method-show-detail.gif)
当我们在编写 Markdown 这样的非编程语言的文档时，也可以通过 `Cmd + 鼠标左键` 来打开超级链接。
![](/images/tech/vs-markdown-open-linked.gif)
