# 升级设置

AirSim 1.2 中的设置架构已更改，以提高灵活性和界面简洁性。如果您有旧版的 [settings.json](settings.md) 文件，您可以删除它并重新启动 AirSim，或者使用本指南进行手动升级。

## 快捷方式

我们建议直接删除 [settings.json](settings.md) 文件，然后重新添加所需的设置。有关可用设置的完整信息，请参阅 [文档](settings.md) 。


## 改变

### UsageScenario

之前我们使用 `UsageScenario` 来指定 `ComputerVision` 模式。现在我们改用 `"SimMode": "ComputerVision"`。

### CameraDefaults 和改变相机设置

之前，我们在根目录中有 `CaptureSettings` 和 `NoiseSettings`。现在，它们被合并到新的 CameraDefaults 元素中。 [此元素的架构](settings.md#camera_settings) 稍后将用于配置车辆上的摄像头。


### Gimbal

现在，[Gimbal 元素](settings.md#Gimbal) （而不是旧的 Gimble 元素）已移出 `CaptureSettings`。

### CameraID 到 CameraName

现在所有设置都通过 [名称](image_apis.md#available_cameras) 而不是 ID 来引用相机。


### 使用 PX4

新的“车辆”元素允许指定要创建的车辆。要使用 PX4，请参阅 [本节](settings.md#using_px4) 。

### AdditionalCameras

旧的`AdditionalCameras`设置现在被车辆设置中的 [Cameras元素](settings.md#Common_Vehicle_Setting) 取代。


