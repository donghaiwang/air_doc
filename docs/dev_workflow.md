# 开发工作流程

以下是如何在使用 AirSim 时执行各种开发活动的指南。如果您是虚幻引擎项目的新手，并且想要为 AirSim 做出贡献或根据自己的定制需求创建自己的分支，这可能会节省您的一些时间。

## 开发环境

### 操作系统
我们强烈推荐使用 Windows 10 和 Visual Studio 2019 作为您的开发环境。遗憾的是，虚幻引擎对其他操作系统和 IDE 的支持并不成熟，如果您尝试变通方法并跳过繁琐的步骤，可能会面临严重的生产力损失。


### 硬件

我们推荐使用 NVidia 1080 或 NVidia Titan 系列等 GPU，搭配性能强大的台式机，例如配备 64GB RAM、6 核及以上处理器、SSD 和 2-3 台显示器（最好是 4K 显示器）的台式机。我们发现 HP Z840 的性能足以满足我们的需求。高端笔记本电脑的开发体验通常不如性能强大的台式机，但在紧急情况下可能会派上用场。通常情况下，您需要配备独立 NVidia GPU（至少是 M2000 或更高版本）、64GB RAM、SSD 以及 4K 显示器的笔记本电脑。我们发现联想 P50 等型号的笔记本电脑能够很好地满足我们的需求。仅配备集成显卡的笔记本电脑可能无法满足我们的需求。


## 更新和更改 AirSim 代码

### 概述
AirSim 被设计为插件形式。这意味着它无法独立运行，需要将其放入 Unreal 项目中（我们称之为“环境”）。因此，构建和测试 AirSim 分为 2 个步骤：(1) 构建插件 (2) 在 Unreal 项目中部署插件并运行该项目。


第一步是通过 AirSim 根目录下的 build.cmd 完成的。此命令将更新 `Unreal\Plugins` 文件夹中插件所需的所有内容。因此，要部署插件，您只需将 `Unreal\Plugins` 文件夹复制到 Unreal 项目文件夹中即可。接下来，您应该删除 Unreal 项目中的所有中间文件，然后为 Unreal 项目重新生成 .sln 文件。为此，我们在`Unreal\Environments\Blocks`文件夹中提供了两个方便的 .bat 文件：`clean.bat`和`GenerateProjectFiles.bat`。因此，只需从 Unreal 项目的根目录按顺序运行这两个 bat 文件即可。现在，您可以在 Visual Studio 中打开新的 .sln 文件并按 F5 运行它。


### 步骤

以下是我们在 AirSim 中进行更改并测试的步骤。在 AirSim 代码中进行开发的最佳方式是使用 [Blocks 项目](unreal_blocks.md)。这是一个轻量级项目，因此编译时间相对较快。通常的工作流程如下：


```
REM //Use x64 Native Tools Command Prompt for VS 2019
REM //Navigate to AirSim repo folder

git pull                          
build.cmd                        
cd Unreal\Environments\Blocks         
update_from_git.bat
start Blocks.sln
```

上述命令首先构建 AirSim 插件，然后使用便捷的`update_from_git.bat`将其部署到 Blocks 项目中。现在，您可以在 Visual Studio 解决方案中工作，修改代码，然后只需按 F5 即可构建、运行并测试更改。调试、断点等功能应该可以正常工作。


完成代码更改后，您可能希望将更改推送回 AirSim 仓库或您自己的 fork，或者您可以将新插件部署到您的自定义 Unreal 项目中。为此，请返回命令提示符并首先更新 AirSim 仓库文件夹：



```
REM //Use x64 Native Tools Command Prompt for VS 2019
REM //run this from Unreal\Environments\Blocks 

update_to_git.bat
build.cmd
```

上述命令会将您的代码更改从 Unreal 项目文件夹转移回`Unreal\Plugins`文件夹。现在，您的更改已准备好推送到 AirSim 代码库或您自己的分支。您也可以将`Unreal\Plugins`复制到您的自定义 Unreal 引擎项目中，并检查您的自定义项目是否一切正常。


### Take Away

一旦你理解了虚幻构建系统和插件模型的工作原理，以及我们为什么要执行上述步骤，你应该就能轻松地遵循这个工作流程了。不必害怕打开 .bat 文件来查看它的内部功能。这些文件非常简洁明了（当然，build.cmd 除外——不要深入研究它）。


## FAQ

#### 我对 Blocks 项目中的代码进行了更改，但它不起作用。
当你在 Visual Studio 中按下 F5 或 F6 开始构建时，虚幻构建系统会启动，并尝试查找是否有任何文件是脏的以及需要构建哪些文件。遗憾的是，它经常无法识别那些不是使用虚幻头文件和对象层次结构的代码的脏文件。所以，诀窍是将某个虚幻构建系统始终能识别的文件设置为脏文件。我最喜欢的文件是 AirSimGameMode.cpp。只需插入一行，删除它并保存文件即可。


#### 我在 Visual Studio 之外对代码进行了更改，但它不起作用。

别这么做！虚幻构建系统*假设*你正在使用 Visual Studio，并且它会执行一系列操作来与 Visual Studio 集成。如果你坚持使用其他编辑器，请查阅如何在虚幻项目中进行命令行构建，或者查看你的编辑器文档，了解如何将其与虚幻构建系统集成，或者运行 `clean.bat` + `GenerateProjectFiles.bat` 以确保 VS 解决方案同步。


#### 我正在尝试在虚幻项目中添加新文件，但没有成功。

不会！虽然您确实在使用 Visual Studio 解决方案，但请记住，该解决方案实际上是由 Unreal Build 系统生成的。如果您想在项目中添加新文件，请先关闭 Visual Studio，在所需位置添加一个空文件，然后运行`GenerateProjectFiles.bat`，它将扫描项目中的所有文件，然后重新创建 .sln 文件。现在打开这个新的 .sln 文件，您就可以开始工作了。


#### 我复制了 Unreal\Plugins 文件夹，但在 Unreal 项目中没有任何反应。

首先确保项目的 .uproject 文件引用了该插件。然后确保已按照上文概述中所述运行 `clean.bat` 和 `GenerateProjectFiles.bat`。


#### 我有多个使用 AirSim 插件的 Unreal 项目。如何轻松更新它们？

你真走运！我们有个`build_all_ue_projects.bat`文件，它正好能完成这个任务。别把它当成黑盒子（至少现在还别），打开它看看它能做什么。它有 4 个变量，这些变量可以通过命令行参数设置。如果未提供这些参数，它们将在下一组语句中被设置为默认值。你可能需要更改路径的默认值。这个批处理文件会构建 AirSim 插件，将其部署到所有列出的项目（请参阅批处理文件后面的 CALL 语句），为这些项目运行打包，并将最终的二进制文件放入指定文件夹——所有操作只需一步！这就是我们用来创建自己的二进制版本的工具。


#### 我如何回馈 AirSim？

在进行任何更改之前，请确保您已创建功能分支。在 Blocks 环境中测试代码更改后，请按照 [常规步骤](https://akrabat.com/the-beginners-guide-to-contributing-to-a-github-project/) 进行贡献，就像其他 GitHub 项目一样。请使用 rebase 和 squash 合并，更多信息请参阅 [Git 合并和 rebase 简介：它们是什么，以及如何使用它们](https://www.freecodecamp.org/news/an-introduction-to-git-merge-and-rebase-what-they-are-and-how-to-use-them-131b863785f/) 。




