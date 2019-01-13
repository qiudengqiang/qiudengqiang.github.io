---
title: Xcode - 代码片段 Code Snippets
date: 2018-05-08 10:01:47
tags: [Xcode]
categories: Xcode
---
在iOS开发过程中，经常会用到一些相似的代码。我们可以将这些代码保存起来，在使用的时候直接从Code Snippets拖拽代码块到指定的位置，也可以设置一些快捷方式来调用Xcode代码片段。
## 新增
* 例如编写以下代码片段
``` objc
@property (nonatomic, strong) <#Type#> *<#value#>;
```
<##> 作用是占位，## 之间可以输入提示文字。
* 使用快捷键：command+option+0
* 将上述代码片段拖拽到下图所示区域
	![](/images/code_snippets.png)
	> **小技巧**：用鼠标选中代码块后把光标放在所选代码块上点击长按2-3秒(光标会由插入标变为小箭头状态)就可以拖拽了

* 弹出下图
	![](/images/code_snippet_dialog.png)
> Title：标题
> Summary：描述
> Platform：可以使用的平台 
> Language：可以在哪些语言中使用
> Completion Shortcut：快捷方式（例如：@xs）。 
> Completion Scopes：作用范围

## 修改
对代码片段进行修改，选中代码片段，点击edit即可。
## 删除
对代码片段进行删除，选中代码片段，按delete键即可。
## Git管理
为了方便在更换电脑后可以更快速的使用自己的代码块，可以托管在Github上进行管理，这样在家里和公司两台Mac任何一端有了更新，另一端随时都可以pull一下使用了。
### Xcode中代码片段默认的目录：
``` bash
~/Library/Developer/Xcode/UserData/CodeSnippets
```
### 同步代码片段
上述目录设置成一个 Git 的版本库，将代码片段放到 Github 上。
