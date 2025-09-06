# Linux 中 Docker 上的 AirSim
我们有 2 种 Docker 选项。您可以构建镜像来运行 AirSim Linux 二进制文件，也可以 [从源代码](#source) 编译虚幻引擎 + AirSim。

## 二进制文件
#### 要求：
- 安装 [nvidia-docker2](https://github.com/NVIDIA/nvidia-docker#quickstart)

#### 构建 docker 镜像
- 以下是默认参数。

  `--base_image`: 这是我们将安装 airsim 的镜像。我们已经在 Ubuntu 18.04 和 CUDA 10.0 上测试过。您可以自行承担风险，指定任何 [NVIDIA cudagl](https://hub.docker.com/r/nvidia/cudagl/) 版本。

   `--target_image` 是你的 Docker 镜像的名称。默认为 `airsim_binary`，标签与基础镜像相同。

```bash
$ cd Airsim/docker;
$ python build_airsim_image.py \
   --base_image=nvidia/cudagl:10.0-devel-ubuntu18.04 \
   --target_image=airsim_binary:10.0-devel-ubuntu18.04
```

- 通过以下方式验证您是否有镜像：
 `$ docker images | grep airsim`

#### 在 Docker 容器内运行虚幻二进制文件
- 获取 [Linux 二进制文件](https://github.com/Microsoft/AirSim/releases) 或在 Ubuntu 中打包你自己的项目。我们以 Blocks 二进制文件为例。你可以运行以下命令下载它：

```bash
   $ cd Airsim/docker;
   $ ./download_blocks_env_binary.sh
```

修改它以获取所需的特定二进制文件。

- 在 Docker 容器内运行虚幻二进制文件。语法如下：

```bash
   $ ./run_airsim_image_binary.sh DOCKER_IMAGE_NAME UNREAL_BINARY_SHELL_SCRIPT UNREAL_BINARY_ARGUMENTS -- headless
```

   对于 Blocks，您可以执行 `$ ./run_airsim_image_binary.sh airsim_binary:10.0-devel-ubuntu18.04 Blocks/Blocks.sh -windowed -ResX=1080 -ResY=720`

   * `DOCKER_IMAGE_NAME`: 与上一步 `target_image` 参数相同，默认输入 `airsim_binary:10.0-devel-ubuntu18.04`
   * `UNREAL_BINARY_SHELL_SCRIPT`: 对于 Blocks 环境，它将是 `Blocks/Blocks.sh`
   * [`UNREAL_BINARY_ARGUMENTS`](https://docs.unrealengine.com/en-us/Programming/Basics/CommandLineArguments):
      对于 airsim 来说，最相关的选项是 `-windowed`, `-ResX`, `-ResY`。点击链接查看所有选项。

  * 无头模式下运行：最后的后缀为 `-- headless`：
```bash
$ ./run_airsim_image_binary.sh Blocks/Blocks.sh -- headless
```

- [指定 `settings.json`](#specifying-settingsjson)

## Source
#### 需要：
- 安装 [nvidia-docker2](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)
- 安装 [ue4-docker](https://docs.adamrehn.com/ue4-docker/configuration/configuring-linux)

#### 在docker中构建虚幻引擎：
- 要访问虚幻引擎的源代码，请在 Epic Games 网站上注册并将其链接到您的 github 帐户，如 [此处](https://docs.unrealengine.com/en-us/Platforms/Linux/BeginnerLinuxDeveloper/SettingUpAnUnrealWorkflow) 的`Required Steps(必需步骤)`部分所述。

    请注意，您不需要执行`步骤 2：在 Linux 上下载 UE4`！

- 构建虚幻引擎 4.19.2 docker 镜像。我们将在示例中使用 CUDA 10.0。
    `$ ue4-docker build 4.19.2 --cuda=10.0 --no-full`
    - [可选] `$ ue4-docker clean` 释放一些空间。 [详情请见此处](https://docs.adamrehn.com/ue4-docker/commands/clean)
    - `ue4-docker` 支持 [这里](https://hub.docker.com/r/nvidia/cudagl/) NVIDIA 的 cudagl dockerhub 上列出的所有 CUDA 版本。
    - 请参阅 [此页面](https://docs.adamrehn.com/ue4-docker/building-images/advanced-build-options) 了解使用 `ue4-docker` 的高级配置

- 磁盘空间：
    - 虚幻的图像和容器会占用大量空间，特别是当您尝试多个版本时。
    - 以下是用于监视 docker 使用的空间并清理中间构建的有用链接列表：
        - [大型容器镜像入门](https://docs.adamrehn.com/ue4-docker/read-these-first/large-container-images-primer)
        - [`docker system df`](https://docs.docker.com/engine/reference/commandline/system_df/)
        - [`docker container prune`](https://docs.docker.com/engine/reference/commandline/container_prune/)
        - [`docker image prune`](https://docs.docker.com/engine/reference/commandline/image_prune/)
        - [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)

#### 在 UE4 docker 容器内构建 AirSim：
* 构建 AirSim docker 镜像（它覆盖我们刚刚构建的虚幻镜像）以下是默认参数。
    - `--base_image`: 这是我们将用于安装 airsim 的镜像。我们已经在 `adamrehn/ue4-engine:4.19.2-cudagl10.0` 上进行了测试。其他版本请参见 [ue4-docker](https://docs.adamrehn.com/ue4-docker/building-images/available-container-images) 。 
    - `--target_image` 是你想要的 Docker 镜像名称。默认为 `airsim_source` ，标签与基础镜像相同。 

```bash
$ cd Airsim/docker;
$ python build_airsim_image.py \
   --source \
   ----base_image adamrehn/ue4-engine:4.19.2-cudagl10.0 \
   --target_image=airsim_source:4.19.2-cudagl10.0
```

#### 运行 AirSim 容器
* 运行我们构建的 airsim 源映像：

```bash
   ./run_airsim_image_source.sh airsim_source:4.19.2-cudagl10.0
```

   语法为 `./run_airsim_image_source.sh DOCKER_IMAGE_NAME -- headless`
   `-- headless`: 后缀此项可在可选的无头模式下运行。

* 在容器内部，您可以在 `/home/ue4` 下看到 `UnrealEngine` 和 `AirSim`。
* 在容器内启动虚幻引擎：
   `ue4@HOSTMACHINE:~$ /home/ue4/UnrealEngine/Engine/Binaries/Linux/UE4Editor`
* [指定 airsim settings.json](#specifying-settingsjson)
* 继续阅读 [AirSim 的 Linux 文档](build_linux.md#build-unreal-environment) 。

#### [杂项] 将虚幻环境打包到 `airsim_source` 容器中
* 我们以 Blocks 环境为例。
    在以下脚本中，`project` 指定 Unreal UProject 文件的完整路径，`archivedirectory` 指定二进制文件所在的目录。

```bash
$ /home/ue4/UnrealEngine/Engine/Build/BatchFiles/RunUAT.sh BuildCookRun -platform=Linux -clientconfig=Shipping -serverconfig=Shipping -noP4 -cook -allmaps -build -stage -prereqs -pak -archive \
-archivedirectory=/home/ue4/Binaries/Blocks/ \
-project=/home/ue4/AirSim/Unreal/Environments/Blocks/Blocks.uproject
```

这将在 `/home/ue4/Binaries/Blocks/` 中创建一个 Blocks 二进制文件。您可以运行 `/home/ue4/Binaries/Blocks/LinuxNoEditor/Blocks.sh -windowed` 来测试它。


### 指定 settings.json
#### `airsim_binary` docker 镜像：
  - 我们将主机的 `PATH/TO/Airsim/docker/settings.json` 映射到 docker 容器的 `/home/airsim_user/Documents/AirSim/settings.json`。 
  - 因此，我们可以通过修改 `PATH_TO_YOUR/settings.json` 来加载任何设置文件，只需修改 [`run_airsim_image_binary.sh`](https://github.com/Microsoft/AirSim/blob/main/docker/run_airsim_image_binary.sh) 中的以下代码片段即可

```bash
nvidia-docker run --runtime=nvidia -it \
      -v $PATH_TO_YOUR/settings.json:/home/airsim_user/Documents/AirSim/settings.json \
      -v $UNREAL_BINARY_PATH:$UNREAL_BINARY_PATH \
      -e SDL_VIDEODRIVER=$SDL_VIDEODRIVER_VALUE \
      -e SDL_HINT_CUDA_DEVICE='0' \
      --net=host \
      --env="DISPLAY=$DISPLAY" \
      --env="QT_X11_NO_MITSHM=1" \
      --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
      -env="XAUTHORITY=$XAUTH" \
      --volume="$XAUTH:$XAUTH" \
      --rm \
      $DOCKER_IMAGE_NAME \
      /bin/bash -c "$UNREAL_BINARY_COMMAND"
```

**注意：** Docker 版本 >=19.03（使用 `docker -v` 检查），原生支持 Nvidia GPU，因此使用 `--gpus all` 标志运行：

```bash
docker run --gpus all -it \
    ...
```

####  `airsim_source` docker 镜像:

  * 我们将主机的 `PATH/TO/Airsim/docker/settings.json` 映射到 docker 容器的 `/home/airsim_user/Documents/AirSim/settings.json`。 
  * 因此，我们可以通过修改 [`run_airsim_image_source.sh`](https://github.com/Microsoft/AirSim/blob/main/docker/run_airsim_image_source.sh) 中的以下代码片段来修改 `PATH_TO_YOUR/settings.json` 来加载任何设置文件：

```bash
   nvidia-docker run --runtime=nvidia -it \
      -v $(pwd)/settings.json:/home/airsim_user/Documents/AirSim/settings.json \
      -e SDL_VIDEODRIVER=$SDL_VIDEODRIVER_VALUE \
      -e SDL_HINT_CUDA_DEVICE='0' \
      --net=host \
      --env="DISPLAY=$DISPLAY" \
      --env="QT_X11_NO_MITSHM=1" \
      --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
      -env="XAUTHORITY=$XAUTH" \
      --volume="$XAUTH:$XAUTH" \
      --rm \
   $DOCKER_IMAGE_NAME
```
