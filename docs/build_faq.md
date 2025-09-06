# FAQ

---

## Windows 构建

* [如何强制 Unreal 使用 Visual Studio 2019？](#how-to-force-unreal-to-use-visual-studio-2019)
* [报错: 'where' is not recognized as an internal or external command](#i-get-error-where-is-not-recognized-as-an-internal-or-external-command)
* [报错：`<MyProject> could not be compiled. Try rebuilding from source manually`](#im-getting-error-myproject-could-not-be-compiled-try-rebuilding-from-source-manually)
* [当运行 build.cmd 时报错：`error C100 : An internal error has occurred in the compiler`](#i-get-error-c100--an-internal-error-has-occurred-in-the-compiler-when-running-buildcmd)
* [报错："'corecrt.h': No such file or directory" or "Windows SDK version 8.1 not found"](#i-get-error-corecrth-no-such-file-or-directory-or-windows-sdk-version-81-not-found)
* [如何将 PX4 固件与 AirSim 一起使用？](#how-do-i-use-px4-firmware-with-airsim)
* [在 Visual Studio 中做了更改，但没有效果](#i-made-changes-in-visual-studio-but-there-is-no-effect)
* [Unreal 仍然使用 VS2015，否则我会收到一些链接错误](#unreal-still-uses-vs2015-or-im-getting-some-link-error)

---

## Linux 构建

* [报错：`<MyProject> could not be compiled. Try rebuilding from source manually`.](#im-getting-error-myproject-could-not-be-compiled-try-rebuilding-from-source-manually)
* [Unreal崩溃了！我怎么知道哪里出了问题？](#unreal-crashed-how-do-i-know-what-went-wrong)
* [如何在 Linux 上使用 IDE？](#how-do-i-use-an-ide-on-linux)
* [我可以从 Windows 机器交叉编译 Linux 吗？](#can-i-cross-compile-for-linux-from-a-windows-machine)
* [AirSim 使用什么编译器和 stdlib？](#what-compiler-and-stdlib-does-airsim-use)
* [AirSim 构建使用哪个版本的 CMake？](#what-version-of-cmake-does-the-airsim-build-use)
* [我可以在 BashOnWindows 中编译 AirSim 吗？](#can-i-compile-airsim-in-bashonwindows)
* [在哪里可以找到有关在 Linux 上运行 Unreal 的更多信息？](#where-can-i-find-more-info-on-running-unreal-on-linux)

---

## 其他

* [打包 AirSim](#packaging-a-binary-including-the-airsim-plugin)

---

<!-- ======================================================================= -->
## Windows 构建
<!-- ======================================================================= -->

###### 如何强制 Unreal 使用 Visual Studio 2019？

>If the default `update_from_git.bat` file results in VS 2017 project, then you may need to run the `C:\Program Files\Epic Games\UE_4.25\Engine\Binaries\DotNET\UnrealBuildTool.exe` tool manually, with the command line options `-projectfiles -project=<your.uproject>  -game -rocket -progress -2019`.
>
>If you are upgrading from 4.18 to 4.25 you may also need to add `BuildSettingsVersion.V2` to your `*.Target.cs` and `*Editor.Target.cs` build files, like this:
>
>```c#
>	public AirSimNHTestTarget(TargetInfo Target) : base(Target)
>	{
>		Type = TargetType.Game;
>		DefaultBuildSettings = BuildSettingsVersion.V2;
>		ExtraModuleNames.AddRange(new string[] { "AirSimNHTest" });
>	}
>```
>
>You may also need to edit this file:
>
>```
>"%APPDATA%\Unreal Engine\UnrealBuildTool\BuildConfiguration.xml
>```
>
>And add this Compiler version setting:
>
>```xml
><Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">
>  <WindowsPlatform>
>    <Compiler>VisualStudio2019</Compiler>
>  </WindowsPlatform>
></Configuration>
>```

<!-- ======================================================================= -->

###### 报错: 'where' is not recognized as an internal or external command

>您必须将 `C:\WINDOWS\SYSTEM32` 添加到您的 PATH 环境变量中。 

<!-- ======================================================================= -->

###### 报错 `<MyProject> could not be compiled. Try rebuilding from source manually`

>当出现编译错误时，就会发生这种情况。日志存储在 `<My-Project>\Saved\Logs` 中，可用于查找问题。 
>
>一个常见问题可能是 Visual Studio 版本冲突，AirSim 使用的是 VS 2019，而 UE 使用的是 VS 2017，可以通过在日志文件中搜索`2017`来找到。在这种情况下，请参阅上面的答案。
>
>如果您修改了 AirSim 插件文件，那么您可以右键单击 `.uproject` 文件，选择`Generate Visual Studio solution file`，然后在 VS 中打开 `.sln` 文件以修复错误并再次构建。

<!-- ======================================================================= -->

###### I get `error C100 : An internal error has occurred in the compiler` when running build.cmd

>We have noticed this happening with VS version `15.9.0` and have checked-in a workaround in AirSim code. If you have this VS version, please make sure to pull the latest AirSim code.

<!-- ======================================================================= -->

###### 报错："'corecrt.h': No such file or directory" or "Windows SDK version 8.1 not found"

>Very likely you don't have [Windows SDK](https://developercommunity.visualstudio.com/content/problem/3754/cant-compile-c-program-because-of-sdk-81cant-add-a.html) installed with Visual Studio.

<!-- ======================================================================= -->

###### 如何将 PX4 固件与 AirSim 一起使用？

>默认情况下，AirSim 使用其内置固件 [simple_flight](simple_flight.md) 。如果您只想使用它，则无需进行其他设置。如果您想改用 PX4，请参阅 [本指南](px4_setup.md) 。 

<!-- ======================================================================= -->

###### 在 Visual Studio 中做了更改，但没有效果

>有时，如果仅更改头文件，Unreal + VS 构建系统可能无法重新编译。为了确保重新编译，请将一些基于 Unreal 的 cpp 文件（例如 AirSimGameMode.cpp）设置为“dirty”。 

<!-- ======================================================================= -->

###### Unreal 仍然使用 VS2015，否则我会收到一些链接错误

>运行多个版本的 VS 可能会导致编译 UE 项目时出现问题。其中一个问题是 UE 会尝试使用旧版本的 VS 进行编译，而旧版本的 VS 可能无法正常工作。虚幻引擎中有两个设置，一个用于引擎，一个用于项目，用于调整要使用的 VS 版本。 
>
>1. Edit -> Editor preferences -> General -> Source code -> Source Code Editor
>2. Edit -> Project Settings -> Platforms -> Windows -> Toolchain ->CompilerVersion
>
>在某些情况下，这些设置仍然不会产生预期的结果，并且可能会产生如下错误：LINK:致命错误 LNK1181:无法打开输入文件“ws2_32.lib” 
>
>为了解决此类问题，可以采用以下步骤：
>
>1. 使用 [VisualStudioUninstaller](https://github.com/Microsoft/VisualStudioUninstaller/releases) 卸载所有旧版本的 VS
>2. Repair/Install VS 2019
>3. 重启机器并安装 Epic 启动器和所需版本的引擎

---

## Linux 构建
<!-- ======================================================================= -->

###### 报错：`<MyProject> could not be compiled. Try rebuilding from source manually`.

>这可能是由于编译错误或你的 gch 文件已过期造成的。查看你的控制台窗口。你看到下面这样的内容了吗？
>
>`fatal error: file '/usr/include/linux/version.h''/usr/include/linux/version.h' has been modified since the precompiled header`
>
>如果是这种情况，请查找该消息后面的 *.gch 文件，删除它们并重试。这是虚幻引擎论坛上的 [相关帖子](https://answers.unrealengine.com/questions/412349/linux-ue4-build-precompiled-header-fatal-error.html) 。
>
>如果您在控制台中看到其他编译错误，请打开这些源文件，看看是否是由于您所做的更改造成的。如果不是，请在 [GitHub](https://github.com/OpenHUTB/air/issues) 上将其报告为问题。 

<!-- ======================================================================= -->

###### Unreal崩溃了！我怎么知道哪里出了问题？

>前往 `MyUnrealProject/Saved/Crashes` 文件夹，并在其子目录中搜索 `MyProject.log` 文件。在该文件末尾，您将看到堆栈跟踪和消息。您还可以查看 `Diagnostics.txt` 文件。 

<!-- ======================================================================= -->

###### 如何在 Linux 上使用 IDE？

>您可以使用 Qt Creator 或 CodeLite。Qt Creator 的使用说明可在 [此处](https://docs.unrealengine.com/en-US/SharingAndReleasing/Linux/BeginnerLinuxDeveloper/SettingUpQtCreator/index.html) 获取。

<!-- ======================================================================= -->

###### 我可以从 Windows 机器交叉编译 Linux 吗？

>是的，可以，但我们还没有测试过。您可以在 [这里](https://docs.unrealengine.com/latest/INT/Platforms/Linux/GettingStarted/index.html) 找到说明。

<!-- ======================================================================= -->

###### AirSim 使用什么编译器和 stdlib？

>我们使用与虚幻引擎相同的编译器 **Clang 8**，以及 stdlib **libc++**。AirSim 的 `setup.sh` 会自动下载它们。

<!-- ======================================================================= -->

###### AirSim 构建使用哪个版本的 CMake？

>3.10.0 或更高版本。这*不是* Ubuntu 16.04 的默认版本，因此 setup.sh 会为您安装。您可以使用 `cmake --version` 检查您的 CMake 版本。如果您使用的是旧版本，请按照 [这些说明](cmake_linux.md) 操作或访问  [CMake 网站](https://cmake.org/install/) 。 

<!-- ======================================================================= -->

###### 我可以在 BashOnWindows 中编译 AirSim 吗？

>是的，但是您无法从 BashOnWindows 运行虚幻引擎。因此，这对于检查 Linux 编译情况很有用，但对于端到端运行则不然。请参阅 [BashOnWindows 安装指南](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide) 。请确保使用最新版本（Windows 10 Creators Edition），因为之前的版本存在各种问题。另外，不要从 `Visual Studio Command Prompt`中调用 `bash`，否则 CMake 可能会找到 VC++ 并尝试使用它！ 

<!-- ======================================================================= -->

###### 在哪里可以找到有关在 Linux 上运行 Unreal 的更多信息？
>从这里开始： [Linux 上的引擎](https://docs.unrealengine.com/latest/INT/Platforms/Linux/index.html)
> 
>[在 Linux 上构建引擎](https://wiki.unrealengine.com/Building_On_Linux#Clang)
> 
>[Unreal 的 Linux 支持](https://wiki.unrealengine.com/Linux_Support)
>
>[引擎交叉编译](https://wiki.unrealengine.com/Compiling_For_Linux)

---

## 其他
<!-- ======================================================================= -->

###### 打包包含 AirSim 插件的二进制文件

>为了将自定义环境与 AirSim 插件打包在一起，需要进行一些项目设置，以确保 AirSim 所需的所有资源都包含在包中。在`Edit -> Project Settings... -> Project -> Packaging`下，请确保正确配置以下设置：
>
>- `List of maps to include in a packaged build`: 确保存在一个`/AirSim/AirSimAssets`条目
>- `Additional Asset Directories to Cook`: 确保存在一个`/AirSim/HUDAssets`条目
