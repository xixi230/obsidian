最适合新手上手的也是最推荐新手上手的编辑器

---
vscode全称Visual Studio Code，来自微软的开源代码编辑器
官方链接：[Visual Studio Code - The open source AI code editor](https://code.visualstudio.com/)
![[Pasted image 20260201092520.png]]

我们需要将他和微软的另一个编辑器区分开Visual Studio
![[Pasted image 20260201092744.png]]
很多同学第一次接触代码编辑器很有可能会安装成下面的Visual Studio，请注意在本科阶段的轻量代码开发我们目前并不推荐臃肿的Visual Studio。

>安装时区别是 VScode的颜色是**蓝色**而非Visual Studio的紫色


## 碎碎念
请尽量从官网下载，否则有概率下到带有捆绑包的恶意软件
![[Pasted image 20260201095221.png]]

### 更新节奏：
[VS Code每月](https://vscode.github.net.cn/updates)发布一个新版本，其中包含新功能和重要的错误修复。大多数平台都支持自动更新，当新版本可用时，系统会提示您安装它。

> 注意：如果你希望按照自己的计划更新 VS Code，则可以[禁用自动更新。](https://vscode.github.net.cn/docs/supporting/faq#_how-do-i-opt-out-of-vs-code-autoupdates)


### 便捷模式
Visual Studio Code 支持[便携式模式](https://en.wikipedia.org/wiki/Portable_application)安装。此模式使 VS Code 创建和维护的所有数据都位于其自身附近，因此可以跨环境移动，例如在 USB 驱动器上。有关详细信息，请参阅[VS Code 便携式模式文档。](https://vscode.github.net.cn/docs/editor/portable)




启动后如图

![[Pasted image 20260201093332.png]]

我们观察页面
这是一个十分现代化的编辑器界面，左侧是功能区(提供核心和拓展功能)，中间是代码编写区，支持vim模式，同时上面的搜索框支持搜索vscode本身的设置。

### 命令面板
根据您当前的上下文访问所有可用命令。

键盘快捷键：Ctrl+Shift+P
命令面板一般会显示在窗口顶部界面
![[Pasted image 20260201141011.png]]


### 快速打开文件
键盘快捷键：`Ctrl+P`
![[Pasted image 20260201142018.png]]

**提示**：输入? 查看命令建议。

### 配置
vscode支持配置文件管理，这意味着你同时可以准备不同的配置的文件随时切换以应对不同的开发环境。配置文件支持导出和导入，这意味着你可以在任意设备上导入你自己的配置文件快速准备开发环境和熟悉的样式。

![[Pasted image 20260201120314.png]]
具体位于左下角的配置栏目


个人资料配置包括：
- 设置 - 在特定于配置文件的`settings.json`文件中。
- 扩展 - 当前配置文件中包含的扩展列表。
- UI 状态 - 视图布局（位置）、可见视图和操作。
- 键绑定 - 在特定于配置文件的`keybindings.json`文件中。
- 片段 - 在特定于配置文件的`{language}.json`文件中。
- 用户任务 - 在特定于配置文件的`tasks.json`文件中。

### 鼠标滚轮
如果你是鼠标用户，vscode支持设置通过`CTRL`+`滚轮`调整字体大小

1. 点击设置或者用默认快捷键`Ctrl` + `,`
2. 在搜索框输入`mouseWheelZoom`
3. 勾选`Editor: Mouse Wheel Zoom`


![[Pasted image 20260201120949.png]]


### 工作区
工作区是vscode特有的概念之一
在vscode窗口实例中，工作区是打开的**一个或者多个文件夹的集合**
在大多数情况下。我们只需要一个文件夹作为工作区。

工作区的概念使 VS Code 能够：
- 配置仅适用于特定文件夹或文件夹但不适用于其他文件夹的设置。
- 保留仅在该工作区上下文中有效的[任务](https://vscode.github.net.cn/docs/editor/tasks)和[调试器启动配置。](https://vscode.github.net.cn/docs/editor/debugging)
- 存储和恢复与该工作区关联的 UI 状态（例如，打开的文件）。
- 仅选择性地启用或禁用该工作区的扩展。


![[Pasted image 20260201142555.png]]

在窗口左上角选中`文件`，同时选中**打开文件夹**
此时VS Code便自动在单文件夹工作区工作




### 终端

VScode 集成了强大的命令行界面
快捷键`CTRL+``

![[Pasted image 20260201150023.png]]

在集成终端你可以使用任意你在计算机安装的shell
一般在windwos默认用的是powershell而不是~~cmd~~
一般在linux默认使用bash

![[Pasted image 20260201150640.png]]
 关于终端的命令见[[讲座/lecture0/shell|shell]]




