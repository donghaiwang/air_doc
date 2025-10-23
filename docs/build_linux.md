# 在 Linux 上构建 AirSim

目前推荐并测试的环境是 **Ubuntu 18.04 LTS**。理论上，你也可以在其他发行版上进行构建，但我们尚未测试。

我们有两个选择 - 您可以在 docker 容器或主机内构建。

## Docker

请参阅 [此处](docker_ubuntu.md) 的说明

## 主机

### 预构建设置

#### 构建虚幻引擎

- 确保你已 [在 Epic Games 注册](https://docs.unrealengine.com/en-US/SharingAndReleasing/Linux/BeginnerLinuxDeveloper/SettingUpAnUnrealWorkflow/index.html) 。这是获取虚幻引擎源代码访问权限的必要条件。

- 将虚幻引擎克隆到您常用的文件夹中并构建（这可能需要一些时间！）。**注意**：我们目前仅支持虚幻引擎 4.27 及以上版本。建议使用 4.27 版本。

```bash
# 前往克隆 GitHub 项目的文件夹
git clone -b 4.27 git@github.com:EpicGames/UnrealEngine.git
cd UnrealEngine
./Setup.sh
./GenerateProjectFiles.sh
make
```

### 构建 AirSim

- 克隆 AirSim 并构建它：

```bash
# 前往克隆 GitHub 项目的文件夹
git clone https://github.com/OpenHUTB/air.git
cd air
```

默认情况下，AirSim 使用 clang 8 进行构建，以兼容 UE 4.27。安装脚本将安装正确版本的 cmake、llvm 和 eigen。

```bash
./setup.sh
./build.sh
# 使用 ./build.sh --debug 以调试模式构建
```

### 构建虚幻环境

最后，您需要一个虚幻项目来托管您的车辆环境。AirSim 自带一个内置的“Blocks 环境”，您可以使用它，也可以创建自己的环境。如果您想设置自己的环境，请参阅 [设置虚幻环境](unreal_proj.md) 。


## 如何使用 AirSim

一旦设置了 AirSim：

- 转到 `UnrealEngine` 安装文件夹并通过运行 `./Engine/Binaries/Linux/UE4Editor` 启动虚幻编辑器。
- 当虚幻引擎提示打开或创建项目时，选择浏览并选择 `AirSim/Unreal/Environments/Blocks`（或您的 [自定义](unreal_custenv.md) 虚幻项目）。
- 或者，可以将项目文件作为命令行参数传递。对于 Blocks：`./Engine/Binaries/Linux/UE4Editor <AirSim_path>/Unreal/Environments/Blocks/Blocks.uproject`
- 如果系统提示您转换项目，请查找“更多选项”或“就地转换”选项。如果系统提示您构建，请选择“是”。如果系统提示您禁用 AirSim 插件，请选择“否”。
- 虚幻编辑器加载后，按播放(Play)按钮。

请参阅 [使用 API](apis.md)  和 [settings.json](settings.md) 了解可用于 AirSim 的各种选项。

!!! 提示
转到“编辑->编辑器首选项”，在“搜索”框中输入“CPU”，并确保未选中“在后台时使用较少的 CPU”(Use Less CPU when in Background)。 

### [可选] 设置遥控器（仅限多旋翼飞行器）

如需手动飞行，则需要遥控器。更多详情，请参阅 [遥控器设置](remote_control.md) 。

或者，您可以使用 [API](apis.md) 进行编程控制，或者使用所谓的 [计算机视觉模式](image_apis.md) 通过键盘移动。

