# 日志查看器

LogViewer 是一款 Windows WPF 应用，用于显示从虚幻模拟器获取的 MavLink 数据流。
您可以使用它来监控无人机飞行过程中的运行情况。
例如，下图显示了模拟器生成的 x、y 和 z 陀螺仪传感器信息的实时图表。


### 使用


您可以打开一个日志文件，它支持 .mavlink 和 PX4 *.ulg 文件，然后您将在左侧的树状视图中看到日志的内容，您选择的任何指标都将添加到右侧。
您可以使用每个图表右上角的小关闭框关闭每个图表，
并且您可以使用顶部工具栏上的“分组图表”按钮将图表分组，使它们共享相同的纵轴。


![Log Viewer](images/log_viewer.png)

还有一个地图选项，可以绘制无人机的 GPS 路径。您还可以加载多个日志文件，以便比较每个日志文件的数据。

### 实时

如果在运行模拟之前连接 LogViewer，您还可以获得实时视图。

![connect](images/log_viewer_connect.png)

为了使其正常工作，您需要使用以下设置配置 `settings.json`：
```
{
    "SettingsVersion": 1.2,
    "SimMode": "Multirotor",
    "Vehicles": {
        "PX4": {
            ...,
            "LogViewerHostIp": "127.0.0.1",
            "LogViewerPort": 14388,
        }
    }
}
```

注意：如果您需要实时 LogViewer 日志记录，请勿使用“Logs”设置。使用“Logs”将日志记录到文件与 LogViewer 日志记录互斥。

只需按下窗口右上角的蓝色连接器按钮，选择“Socket”选项卡，输入端口号 `14388` 和您的 `localhost` 网络即可。如果您在 Windows 上使用 WSL 2，请选择 `vEthernet (WSL)`。

如果您确实选择了 `vEthernet (WSL)`，那么请确保您还将 `LocalHostIp` 和 `LogViewerHostIp` 设置为匹配的 WSL 以太网地址，例如 `172.31.64.1`。

然后按下记录按钮（工具栏右侧的三角形）。现在启动模拟器，数据将开始流入 LogViewer。

日志查看器中的无人机视图显示了来自 PX4 的实际估算位置，因此这是检查 PX4 是否与模拟器同步的好方法。有时，随着姿态估算与实际情况的同步，您可能会看到一些漂移，在发生严重碰撞后，这种情况会更加明显。

### 安装

如果您无法构建 LogViewer.sln，还可以 [单击一次安装程序。](https://lovettsoftwarestorage.blob.core.windows.net/downloads/Px4LogViewer/Px4LogViewer.application) 。


### 配置

您可以通过编辑 [settings.json 文件) 在模拟器中配置魔术端口号 14388。如果您在 LogViewer 连接对话框中更改了端口号，请确保在 `settings.json` 文件中进行相应的更改。

### 调试

有关如何使用 LogViewer 调试您正在设置的情况的更多信息，请参阅 [PX4 日志记录](px4_logging.md) 。

