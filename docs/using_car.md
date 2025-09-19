# 如何在 AirSim 中使用汽车

默认情况下，AirSim 会提示用户选择要使用的车辆。您可以通过设置 [SimMode](settings.md#SimMode) 轻松更改此设置。例如，如果您想使用汽车，只需在 [settings.json](settings.md)（位于 `~/Documents/AirSim` 文件夹中）中设置 SimMode 即可，如下所示：

```
{
  "SettingsVersion": 1.2,
  "SimMode": "Car"
}
```

现在，当您重新启动 AirSim 时，您应该会看到汽车自动生成。

## 手动驾驶

请使用键盘方向键进行手动驾驶。空格键用于拉手刹。手动驾驶模式下，档位设置为“自动(auto)”。

## 使用 APIs

您可以通过调用各种客户端语言（包括 C++ 和 Python）的 API 来控制车辆、获取状态和图像。更多详情，请参阅 [API 文档](apis.md) 。


## 改变视角

默认情况下，摄像头会从车辆后方追击。您可以按 `F` 键进入 FPV 视图，然后按 `/` 键切换回后方追击。按 F1 键可以查看更多键盘快捷键。


## 相机

车辆默认安装有 5 个摄像头：中央摄像头、左侧摄像头、右侧摄像头、驾驶员摄像头和倒车摄像头。您可以通过指定 [名称](image_apis.md#available_cameras) 来选择这些摄像头的图像。

