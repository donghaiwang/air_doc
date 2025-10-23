# Azure 上的 AirSim 开发环境

本文档介绍如何在 Azure 上自动创建开发环境，以及如何使用 Visual Studio Code 编写和调试连接到 AirSim 的 Python 应用程序

## 自动部署您的 Azure VM

点击蓝色按钮开始 Azure 部署（模板中已预先填充了以下 2 个教程用例的推荐虚拟机大小）


<a href="https://aka.ms/AA8umgt" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>  
**注意**：虚拟机部署和配置过程可能需要 20 多分钟才能完成

### 关于 Azure VM 的部署
- 使用 Azure 试用帐户时，默认 vCPU 配额不足以分配运行 AirSim 所需的虚拟机。如果出现这种情况，您将在尝试创建虚拟机时看到错误，并且需要提交增加配额的请求。**请务必了解虚拟机的使用方式和费用**。

- 为避免在不使用时对虚拟机使用产生费用，请记住从 [Azure 门户](https://portal.azure.com) 释放其资源或从 Azure CLI 使用以下命令：
```bash
az vm deallocate --resource-group MyResourceGroup --name MyVMName
```

## 通过 Visual Studio Code 和远程 SSH 进行编码和调试
- 安装 Visual Studio Code
- 安装 *Remote - SSH* 扩展
- 按 `F1` 并运行 `Remote - SSH: Connect to host...` 命令
- 添加最近创建的虚拟机详细信息。例如， `AzureUser@11.22.33.44`
- 再次运行 `Remote - SSH: Connect to host...` 命令，现在选择新添加的连接。
- 连接后，点击 Visual Studio Code 中的`Clone Repository`按钮，然后将其克隆到远程虚拟机中并*仅打开 `azure` 文件夹*，或者创建一个全新的存储库，克隆它并将 `azure` 文件夹的内容从此存储库复制到其中。务必打开该目录，以便 Visual Studio Code 能够针对具体场景使用特定的 `.vscode` 目录，而不是通用的 AirSim `.vscode` 目录。该目录包含建议安装的扩展程序、远程启动 AirSim 的任务以及 Python 应用程序的启动配置。
- 安装所有推荐的扩展
- 按 `F1` 并选择`Tasks: Run Task`选项。然后，从 Visual Studio Code 中选择`Start AirSim`任务，以从 Visual Studio Code 执行 `start-airsim.ps1` 脚本。
- 打开`app`目录中的 `multirotor.py` 文件
- 开始使用 Python 进行调试
- 完成后，请记住停止并解除分配 Azure VM，以避免产生额外费用

## 从本地 Visual Studio Code 进行编码和调试，并通过转发端口连接到 AirSim

!!! 注意
    此场景将使用 2 个 Visual Studio Code 实例。第一个实例将用作桥梁，通过 SSH 将端口转发到 Azure VM 并执行远程进程；第二个实例将用于本地 Python 开发。为了能够从本地 Python 代码访问 VM，需要保持 Visual Studio Code 的 `Remote - SSH` 实例处于打开状态，同时在第二个实例上使用本地 Python 环境。


- 打开第一个 Visual Studio Code 实例
- 按照上一节中的步骤通过 `Remote - SSH` 进行连接 
- 在 *远程资源管理器(Remote Explorer)*中，将端口 `41451` 添加为本地主机的转发端口
- 按照上一场景中的说明，使用远程会话在 Visual Studio Code 上运行`Start AirSim`任务，或者在 VM 中手动启动 AirSim 二进制文件
- 打开第二个 Visual Studio Code 实例，无需断开或关闭第一个实例
- 要么在本地克隆此存储库并**仅**在 Visual Studio Code 中打开 `azure` **文件夹**，要么创建一个全新的存储库，克隆它并从该存储库中复制 `azure` 文件夹的内容。
- 在 `app` 目录中运行 `pip install -r requirements.txt` 
- 打开 `app` 目录中的 `multirotor.py` 文件
- 开始使用 Python 进行调试
- 完成后，请记住停止并解除分配 Azure VM，以避免产生额外费用

## 使用 Docker 运行

一旦 AirSim 环境和 Python 应用程序准备就绪，您就可以将所有内容打包为 Docker 镜像。`azure` 目录中的示例项目已准备好使用 Docker 运行预构建的 AirSim 二进制文件和 Python 代码。


当你想要大规模运行模拟时，这将是一个完美的方案。例如，你可以为同一个模拟设置几种不同的配置，并使用 Azure 容器服务上的 Docker 镜像以并行、无人值守的方式执行它们。


由于 AirSim 需要访问主机 GPU，因此需要使用支持该功能的 Docker 运行时。有关在 Docker 中运行 AirSim 的更多信息，请点击 [此处](docker_ubuntu.md) 。


使用 Azure 容器服务运行此映像时，唯一的额外要求是向将要部署的容器组添加 GPU 支持。


它可以使用来自 DockerHub 的公共 docker 镜像或部署到私有 Azure 容器注册表的镜像。


### 构建 docker 镜像

```bash
docker build -t <your-registry-url>/<your-image-name> -f ./docker/Dockerfile .`
```

## 使用不同的 AirSim 二进制文件

要使用不同的 AirSim 二进制文件，请首先查看官方文档：[如何在 Windows 上构建 AirSim](build_windows.md)  以及 [如何在 Linux 上构建 AirSim](build_linux.md) （如果您还想使用 Docker 运行它）


一旦您拥有包含新 AirSim 环境的 zip 文件（或者更喜欢使用 [官方版本](https://github.com/OpenHUTB/air/releases) 中的文件），您需要修改存储库的 `azure` 目录中的某些脚本以指向新环境：
- 在 [`azure/azure-env-creation/configure-vm.ps1`](https://github.com/microsoft/AirSim/blob/main/azure/azure-env-creation/configure-vm.ps1) 中，使用新值修改以 `$airSimBinary` 开头的所有参数
- 在 [`azure/start-airsim.ps1`](https://github.com/microsoft/AirSim/blob/main/azure/start-airsim.ps1) 中，使用新值修改 `$airSimExecutable` 和 `$airSimProcessName`


如果你使用的是docker镜像，你还需要一个linux二进制zip文件，并修改以下Docker相关的文件：
- 在 [`azure/docker/Dockerfile`](https://github.com/microsoft/AirSim/blob/main/azure/docker/Dockerfile) 中，使用新值修改 `AIRSIM_BINARY_ZIP_URL` 和 `AIRSIM_BINARY_ZIP_FILENAME` ENV 声明
- 在 [`azure/docker/docker-entrypoint.sh`](https://github.com/microsoft/AirSim/blob/main/azure/docker/docker-entrypoint.sh) 中，使用新值修改 `AIRSIM_EXECUTABLE`


## 维护此开发环境

此开发环境的多个组件（ARM 模板、初始化脚本和 VSCode 任务）直接依赖于当前目录结构、文件名和存储库位置。计划 修改/fork 其中任何一个组件时，请务必检查每个脚本和模板，以进行必要的调整。
