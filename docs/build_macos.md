# 在 macOS 上构建 AirSim

目前仅测试了 **Catalina (10.15)** 。理论上，AirSim 应该可以在更高版本的 macOS 和 Apple Silicon 硬件上运行，但此路径尚未得到官方支持。

我们有两个选择 - 您可以在 docker 容器或主机内构建。

## Docker

请参阅 [此处](docker_ubuntu.md) 的说明


## 主机

### 预构建设置

#### 下载虚幻引擎

1. [下载](https://www.unrealengine.com/download) Epic Games 启动器。虽然虚幻引擎是开源的，可以免费下载，但仍然需要注册。
2. 运行 Epic Games 启动器，打开左侧窗格中的`Library`选项卡。点击`Add Versions`，此时将显示下载**虚幻 4.27** 的选项，如下所示。如果您安装了多个版本的虚幻，请确保通过点击相应版本“启动”按钮旁边的向下箭头**将 4.27 设置为当前版本**。

   **注意**: AirSim 也适用于 UE 4.24 及以上版本，但我们推荐使用 4.27 版本。

   **注意**: 如果您拥有 UE 4.16 或更早版本的项目，请参阅 [升级指南](unreal_upgrade.md) 来升级您的项目。

### 构建 AirSim

- 克隆 AirSim 并构建它：

```bash
# 前往克隆 GitHub 项目的文件夹
git clone https://github.com/OpenHUTB/air.git
cd air
```

默认情况下，AirSim 使用 clang 8 进行构建，以兼容 UE 4.25。安装脚本将安装正确版本的 cmake、llvm 和 eigen。

在 Apple Silicon 上构建需要 CMake 3.19.2 。

```bash
./setup.sh
./build.sh
# 使用 ./build.sh --debug 在调试模式下构建
```

### 构建虚幻环境

最后，您需要一个虚幻项目来托管您的载具环境。AirSim 自带一个内置的“Blocks 环境”，您可以使用它，也可以创建自己的环境。如果您想设置自己的环境，请参阅 [设置虚幻环境](unreal_proj.md) 。

## 如何使用 AirSim

- 浏览到 `AirSim/Unreal/Environments/Blocks`.
- 从终端运行 `./GenerateProjectFiles.sh <UE_PATH>` ，其中 `UE_PATH` 是虚幻引擎安装文件夹的路径。（默认情况下，该路径为 `/Users/Shared/Epic\ Games/UE_4.27/`）该脚本会创建一个名为 Blocks.xcworkspace 的 XCode 工作区。
- 打开 XCode 工作区，然后按左上角的“构建并运行”按钮。
- 虚幻编辑器加载后，按播放按钮。

请参阅 [使用 API](apis.md)  和 [settings.json](settings.md) 了解可用于 AirSim 的各种选项。

!!! 提示
转到“编辑->编辑器首选项”，在“搜索”框中输入“CPU”，并确保未选中“在后台时使用较少的 CPU”(Use Less CPU when in Background)。

### [可选] 设置遥控器（仅限多旋翼飞行器） 

如需手动飞行，则需要遥控器。更多详情，请参阅 [遥控器设置](remote_control.md) 。

或者，您可以使用 [APIs](apis.md) 进行编程控制，或者使用所谓的 [计算机视觉模式](image_apis.md) 通过键盘移动。

