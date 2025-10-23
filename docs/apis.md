# AirSim APIs

## 介绍

AirSim 提供 API，方便您以编程方式与模拟中的车辆进行交互。您可以使用这些 API 来检索图像、获取状态、控制车辆等等。


## Python 快速入门
如果您想使用 Python 调用 AirSim API，我们建议使用带有 Python 3.5 或更高版本的 Anaconda，但某些代码也可能适用于 Python 2.7（ [帮助我们](CONTRIBUTING.md) 提高兼容性！）。

首先安装 Python 虚拟环境：
```shell
conda create -n air python=3.8
conda activate air
```

首先安装这个包：

```
pip install msgpack-rpc-python
```

您可以从 [releases](https://github.com/OpenHUTB/air/releases) 获取 AirSim 二进制文件，也可以从源代码编译（[Windows](build_windows.md)、[Linux](build_linux.md)）。一旦您可以运行 AirSim，请选择 Car 作为载具，然后导航到 `PythonClient\car\` 文件夹并运行：

```
python hello_car.py
```

如果您正在使用 Visual Studio 2019，则只需打开 AirSim.sln ，将 PythonClient 设置为启动项目，并选择 `car\hello_car.py` 作为启动脚本。


### 安装 AirSim 包

您也可以简单地通过以下方式安装 `airsim` 包：，

```
pip install airsim
```

您可以在代码仓库的 `PythonClient` 文件夹中找到该包的源代码和示例。

!!! 笔记
    1. 您可能会注意到在我们的示例文件夹中有一个 `setup_path.py` 文件。这个文件有一个简单的代码来检测 `airsim` 包是否在父文件夹中可用，在这种情况下，我们使用它而不是 pip 安装的包，所以您总是使用最新的代码。
    2. [低空项目](https://github.com/OpenHUTB/air) 仍在大量开发中，这意味着您可能需要经常更新软件包以使用新的 API。

## C++ 用户

如果要使用 C++ 的 API 和示例，请参阅 [C++ APIs 指南](apis_cpp.md) 。


## Hello Drone
下面介绍了如何使用 Python 的 AirSim API 来控制模拟四旋翼飞行器，准备运行示例 [hello_drone.py](https://github.com/OpenHUTB/air/blob/main/PythonClient/multirotor/hello_drone.py)（另请参阅 [ C++ 示例](apis_cpp.md#hello_drone)）：

```python
import airsim
import os

# 连接到 AirSim 模拟器
client = airsim.MultirotorClient()
client.confirmConnection()
client.enableApiControl(True)
client.armDisarm(True)

# Async 方法返回 Future。调用 join() 等待任务完成。
client.takeoffAsync().join()
client.moveToPositionAsync(-10, 10, -10, 5).join()

# 获取图像
responses = client.simGetImages([
    airsim.ImageRequest("0", airsim.ImageType.DepthVis),
    airsim.ImageRequest("1", airsim.ImageType.DepthPlanar, True)])
print('Retrieved images: %d', len(responses))

# 对图像执行一些操作
for response in responses:
    if response.pixels_as_float:
        print("Type %d, size %d" % (response.image_type, len(response.image_data_float)))
        airsim.write_pfm(os.path.normpath('/temp/py1.pfm'), airsim.get_pfm_array(response))
    else:
        print("Type %d, size %d" % (response.image_type, len(response.image_data_uint8)))
        airsim.write_file(os.path.normpath('/temp/py1.png'), response.image_data_uint8)
```

## Hello Car

下面是如何使用 Python 的 AirSim API 控制模拟汽车（另请参见 [C++ 示例](apis_cpp.md#hello_car) ）：


```python
# 准备运行示例：PythonClient/car/hello_car.py
import airsim
import time

# 连接到 AirSim 模拟器
client = airsim.CarClient()
client.confirmConnection()
client.enableApiControl(True)
car_controls = airsim.CarControls()

while True:
    # 获取车辆状态
    car_state = client.getCarState()
    print("Speed %d, Gear %d" % (car_state.speed, car_state.gear))

    # 控制车辆
    car_controls.throttle = 1
    car_controls.steering = 1
    client.setCarControls(car_controls)

    # 让汽车开一会儿
    time.sleep(1)

    # 从汽车中获取相机图像
    responses = client.simGetImages([
        airsim.ImageRequest(0, airsim.ImageType.DepthVis),
        airsim.ImageRequest(1, airsim.ImageType.DepthPlanar, True)])
    print('Retrieved images: %d', len(responses))

    # 对图像执行一些操作
    for response in responses:
        if response.pixels_as_float:
            print("Type %d, size %d" % (response.image_type, len(response.image_data_float)))
            airsim.write_pfm('py1.pfm', airsim.get_pfm_array(response))
        else:
            print("Type %d, size %d" % (response.image_type, len(response.image_data_uint8)))
            airsim.write_file('py1.png', response.image_data_uint8)
```


## 常用 API

* `reset`: 这会将车辆重置到其原始启动状态。请注意，调用 reset 后必须再次调用 `enableApiControl` 和 `armDisarm` 。
* `confirmConnection`: 每 1 秒检查一次连接状态并在控制台中报告，以便用户可以看到连接进度。
* `enableApiControl`: 出于安全考虑，自动驾驶车辆的 API 控制默认未启用，人类操作员拥有完全控制权（通常通过 RC 或模拟器中的操纵杆）。客户端必须发出此调用才能通过 API 请求控制。车辆的人类操作员可能已禁用 API 控制，这意味着 enableApiControl 无效。可以通过`isApiControlEnabled`检查这一点。 
* `isApiControlEnabled`: 如果已建立 API 控制，则返回 true。如果返回 false（默认值），则 API 调用将被忽略。成功调用 `enableApiControl` 后，`isApiControlEnabled` 应该返回 true。
* `ping`: 如果连接建立，则此调用将返回 true，否则将被阻止直至超时。
* `simPrintLogMessage`: 在模拟器窗口中打印指定消息。如果同时提供了 message_param，则该消息会打印在消息旁边。在这种情况下，如果再次使用相同的消息值但不同的 message_param 调用此 API，则上一行将被新行覆盖（而不是 API 在显示屏上创建新行）。例如，当使用不同的 i 值调用 API 时，`simPrintLogMessage("Iteration: ", to_string(i))` 会持续更新显示屏上的同一行。严重性参数的有效值为 0 到 3（含），分别对应不同的颜色。 
* `simGetObjectPose`, `simSetObjectPose`: 获取并设置虚幻环境中指定对象的姿势。此处，对象在虚幻术语中表示"actor"。它们通过标签和名称进行搜索。请注意，虚幻编辑器中显示的名称是每次运行时 *自动生成的* ，并非永久的。因此，如果您想通过名称引用actor，则必须在虚幻编辑器中更改其自动生成的名称。或者，您可以为actor添加标签，方法是在虚幻编辑器中点击该actor，然后转到 [Tags 属性](https://answers.unrealengine.com/questions/543807/whats-the-difference-between-tag-and-tag.html) ，点击“+”号并添加一些字符串值。如果多个actor具有相同的标签，则返回第一个匹配的actor。如果没有找到匹配项，则返回NaN姿势。返回的姿势以世界坐标系中的NED坐标系表示，采用SI单位。对于`simSetObjectPose`，指定的actor必须将 [Mobility](https://docs.unrealengine.com/en-us/Engine/Actors/Mobility) 设置为Movable，否则将导致未定义的行为。`simSetObjectPose`具有参数`teleport`，表示对象在其路径上 [移动穿过其他对象](https://www.unrealengine.com/en-US/blog/moving-physical-objects) ，如果移动成功，则返回true。 

### 图像 / 计算机视觉 APIs

AirSim 提供全面的图像 API，用于检索来自多个摄像头的同步图像以及包括深度、视差、表面法线和视觉在内的地面实况。您可以在 [settings.json](settings.md) 中设置分辨率、FOV、运动模糊等参数。此外，还提供了用于检测碰撞状态的 API。另请参阅 [完整代码](https://github.com/Microsoft/AirSim/tree/main/Examples/DataCollection/StereoImageGenerator.hpp) ，该代码生成指定数量的立体图像和地面实况深度，并根据相机平面进行归一化、计算视差图像并将其保存为 [pfm 格式](pfm.md) 。


更多关于 [图像 API 和计算机视觉模式的信息](image_apis.md) 。对于可以从域随机化中获益的视觉问题，还有一个 [object retexturing API](retexturing.md) ，可在支持的场景中使用。


### 暂停和继续 API

AirSim 允许通过 `pause(is_paused)` API 暂停和继续模拟。要暂停模拟，请调用 `pause(True)` ；要继续模拟，请调用 `pause(False)` 。您可能会遇到一些情况，尤其是在使用强化学习时，需要模拟运行指定的时间后自动暂停。在模拟暂停期间，您可以进行一些高开销的计算，发送新命令，然后再次运行模拟指定的时间。这可以通过 API `continueForTime(seconds)` 实现。此 API 会运行模拟指定的秒数，然后暂停模拟。有关用法示例，请参阅 [pause_continue_car.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car/pause_continue_car.py) 和 [pause_continue_drone.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor/pause_continue_drone.py) 。


### 碰撞 API

可以使用 `simGetCollisionInfo` API 获取碰撞信息。此调用返回一个结构体，该结构体不仅包含碰撞是否发生的信息，还包含碰撞位置、表面法线、穿透深度等信息。


### 时间 API

AirSim 假设您的环境中存在 `EngineSky/BP_Sky_Sphere` 类的天空球体，并且包含 [ADirectionalLight actor](https://github.com/microsoft/AirSim/blob/v1.4.0-linux/Unreal/Plugins/AirSim/Source/SimMode/SimModeBase.cpp#L224) 。默认情况下，场景中的太阳位置不会随时间移动。您可以使用 [设置](settings.md#timeofday) 来设置 AirSim 用于计算场景中太阳位置的纬度、经度、日期和时间。

您还可以使用以下 API 调用根据给定的日期时间设置太阳位置：

```
simSetTimeOfDay(self, is_enabled, start_datetime = "", is_start_datetime_dst = False, celestial_clock_speed = 1, update_interval_secs = 60, move_sun = True)
```

`is_enabled` 参数必须为 `True` 才能启用时间效果。如果为 `False`，则太阳位置将重置为环境中的原始位置。

其它参数与 [设置](settings.md#timeofday) 相同。


### 视线和世界范围 API

要测试模拟器中从车辆到某个点或两点之间的视线，请分别使用 `simTestLineOfSightToPoint(point, vehicle_name)` 和 `simTestLineOfSightBetweenPoints(point1, point2)` 函数。模拟器的世界范围以两个 `GeoPoints` 向量的形式表示，可以使用 `simGetWorldExtents()` 函数获取。

### 天气 API

默认情况下，所有天气效果均已禁用。要启用天气效果，请先调用：

```
simEnableWeather(True)
```

可以使用 `simSetWeatherParameter` 方法启用各种天气效果，该方法采用 `WeatherParameter`，例如，


```
client.simSetWeatherParameter(airsim.WeatherParameter.Rain, 0.25);
```

第二个参数值从0到1。第一个参数提供以下选项：

```
class WeatherParameter:
    Rain = 0
    Roadwetness = 1
    Snow = 2
    RoadSnow = 3
    MapleLeaf = 4
    RoadLeaf = 5
    Dust = 6
    Fog = 7
```

请注意，湿润道路`Roadwetness`、路上积雪 `RoadSnow` 和 路上叶子`RoadLeaf` 效果需要在场景中添加 [材质](https://github.com/OpenHUTB/air/tree/main/Unreal/Plugins/AirSim/Content/Weather/WeatherFX) 。

请参阅 [示例代码](https://github.com/OpenHUTB/air/blob/main/PythonClient/environment/weather.py) 以了解更多详细信息。


### 记录 APIs


记录 API 可用于通过 API 开始记录数据。可以使用 [设置](settings.md#recording) 指定要记录的数据。要开始记录，请使用：

```
client.startRecording()
```

类似地，要停止记录，请使用 `client.stopRecording()`。要检查记录是否正在运行，请调用 `client.isRecording()`，返回一个布尔值`bool`。

此 API 与使用 `R` 键切换记录功能配合使用，因此，如果使用 `R` 键启用记录功能，`isRecording()` 将返回 `True`，并且可以通过 API 使用 `stopRecording()` 停止记录。同样，如果在视口中按下 `R` 键，使用 API 启动的录制也会停止。如果使用 API 启动或停止录制，LogMessage 也会显示在视口的左上角。

请注意，这只会保存设置中指定的数据。为了更自由地存储数据（例如某些传感器信息），或以其他格式或布局存储数据，请使用其他 API 获取数据并根据需要保存。有关如何修改正在记录的运动学数据的详细信息，请参阅 [修改记录数据](modify_recording_data.md) 。


### 风 API

可以使用 `simSetWind()` 在模拟过程中更改风向。风向以世界坐标系、NED 方向和米/秒为单位指定。

例如，要设置北（前进）方向的风速为 20 米/秒 ：

```python
# 将风向设置为 NED（前进方向）的 (20,0,0)
wind = airsim.Vector3r(20, 0, 0)
client.simSetWind(wind)
```

另请参阅 [set_wind.py](https://github.com/OpenHUTB/air/blob/main/PythonClient/multirotor/set_wind.py) 中的示例脚本


### 激光雷达 API

AirSim 提供 API 来检索车载激光雷达传感器的点云数据。您可以在 [settings.json](settings.md) 中设置通道数、每秒点数、水平和垂直视场等参数。

有关 [激光雷达 API 和设置](lidar.md) 以及 [传感器设置](sensors.md) 的更多信息

### 灯光控制 APIs

可以通过 `simSpawnObject()` API 创建可在 AirSim 内部操控的光源，只需将 `PointLightBP` 或 `SpotLightBP` 作为 `asset_name` 参数传递，并将 `True` 作为 `is_blueprint` 参数传递即可。灯光生成后，即可使用以下 API 进行操控：

* `simSetLightIntensity`: 这允许您编辑光源的强度或亮度。它接受两个参数：`light_name`（上次调用 `simSpawnObject()` 返回的光源对象的名称）和 `intensity`（浮点值）。

### 纹理 API

可以通过以下 API 在对象上动态设置纹理：

* `simSetObjectMaterial`: 此函数使用现有的虚幻材质资源设置对象的材质。它接受两个字符串参数：`object_name` 和 `material_name` 。
* `simSetObjectMaterialFromTexture`: 此函数使用纹理路径设置对象的材质。它接受两个字符串参数：`object_name` 和 `texture_path`。 

### 多载具

AirSim 支持多载具并通过 API 进行控制。请参阅 [多载具](multi_vehicle.md) 文档。


### 坐标系统

所有 AirSim API 均使用 NED 坐标系，即 +X 代表北、+Y 代表东、+Z 代表下。所有单位均为 SI 系统。请注意，这与虚幻引擎内部使用的坐标系不同。在虚幻引擎中，+Z 代表上而不是下，长度单位是厘米而不是米。AirSim API 负责相应的转换。车辆的起点在 NED 系统中始终是坐标 (0, 0, 0)。因此，当从虚幻坐标转换为 NED 时，我们首先减去起始偏移量，然后按比例缩放 100 以进行厘米到米的转换。车辆在放置 Player Start 组件的虚幻环境中生成。 [settings.json](settings.md) 中有一个名为 `OriginGeopoint` 的设置，它为 Player Start 组件分配地理经度、经度和海拔。


## 载具特定 API

### 汽车 API

汽车有以下可用的 API：

* `setCarControls`: 这使您可以设置油门、转向、手刹和自动档或手动档。 
* `getCarState`: 这将检索状态信息，包括速度、当前档位以及 6 个运动学量：位置、方向、线速度和角速度、线加速度和角加速度。所有量均采用 NED 坐标系，除角速度和加速度采用体坐标系外，其他量均采用世界坐标系的 SI 单位。
* [图像 API](image_apis.md).

### 多旋翼 API
多旋翼飞行器可以通过指定角度、速度矢量、目标位置或这些参数的组合来控制。有相应的 `move*` API 来实现此目的。进行位置控制时，我们需要使用某种路径跟随算法。AirSim 默认使用胡萝卜跟随算法。这通常被称为“高层控制”，因为您只需指定高级目标，其余部分由固件处理。目前 AirSim 中可用的最低层控制是 `moveByAngleThrottleAsync` API。


#### getMultirotorState

此 API 一次调用即可返回飞行器的状态。状态包括碰撞、估计运动学（即通过融合传感器计算的运动学）和时间戳（自纪元以来的纳秒数）。此处的运动学包含六个物理量：位置、方向、线速度和角速度、线加速度和角加速度。请注意，simple_slight 当前不支持状态估计器，这意味着对于 simple_flight，估计运动学值和地面真实运动学值将相同。不过，PX4 可以使用除角加速度之外的估计运动学值。所有物理量均采用 NED 坐标系，除角速度和加速度采用机身坐标系外，其他物理量均采用世界坐标系的 SI 单位。


#### 异步方法、持续时间和 max_wait_seconds

许多 API 方法都包含名为`duration`或`max_wait_seconds`的参数，并以*Async*作为后缀，例如 `takeoffAsync`。这些方法在 AirSim 中启动任务后会立即返回，以便您的客户端代码可以在该任务执行期间执行其他操作。如果您想等待此任务完成，可以像这样调用 `waitOnLastTask`：

```cpp
//C++
client.takeoffAsync()->waitOnLastTask();
```

```cpp
# Python
client.takeoffAsync().join()
```

如果您启动另一个命令，它会自动取消上一个任务并启动新的命令。这允许您使用一种模式，您的代码会持续进行感知，计算新的轨迹，并在 AirSim 中将该路径发送给车辆。每个新发出的轨迹都会取消之前的轨迹，从而使您的代码能够在新的传感器数据到达时持续进行更新。

所有 *异步* 方法在 Python 中返回的是 `concurrent.futures.Future`（在 C++ 中返回的是 `std::future`）。请注意，这些 Future 类目前不支持检查状态或取消任务；它们只允许等待任务完成。不过，AirSim 提供了 `cancelLastTask` 接口。


#### drivetrain

飞行器有两种飞行模式：`drivetrain`参数设置为 `airsim.DrivetrainType.ForwardOnly` 或 `airsim.DrivetrainType.MaxDegreeOfFreedom`。如果指定 ForwardOnly，则表示飞行器的前端应始终指向行进方向。因此，如果希望无人机左转，则无人机首先会旋转，使前端指向左侧。此模式在只有前置摄像头并使用 FPV 视图操作飞行器时非常有用。这或多或少就像开车旅行，您始终能看到前方视野。MaxDegreeOfFreedom 意味着您不必关心前端指向何处。因此，左转时，您就像螃蟹一样开始向左转。四旋翼飞行器可以朝任何方向飞行，无论前端指向何处。MaxDegreeOfFreedom 启用此模式。


#### yaw_mode

`yaw_mode` 是一个 `YawMode` 结构体，包含两个字段：`yaw_or_rate` 和 `is_rate`。如果 `is_rate` 字段为 `True`，则 `yaw_or_rate` 字段将被解析为以度/秒为单位的角速度，这意味着您希望车辆在移动时以该角速度绕其轴线连续旋转。如果 `is_rate` 为 `False`，则 `yaw_or_rate` 将被解析为以度为单位的角度，这意味着您希望车辆旋转到特定角度（即偏航角），并在移动时保持该角度。


您可能已经注意到，当 `yaw_mode.is_rate == true` 时，`drivetrain`参数不应设置为 `ForwardOnly`，因为您的说法与“保持前部指向前方但同时持续旋转”的说法相矛盾。但是，如果您在 `ForwardOnly` 模式下将 `yaw_mode.is_rate = false`，则可以实现一些更炫酷的功能。例如，您可以让无人机绕圈飞行，并将 yaw_or_rate 设置为 90，这样相机始终指向中心（“超酷的自拍模式”）。在 `MaxDegreeofFreedom` 中，您也可以通过设置 `yaw_mode.is_rate = true` 并设置 `yaw_mode.yaw_or_rate = 20` 来实现一些更炫酷的功能。这将使无人机在旋转的同时沿其路径飞行，从而可以进行 360 度扫描。


在大多数情况下，您只是不希望偏航发生改变，您可以通过将偏航率设置为 0 来实现。此方法的简写为 `airsim.YawMode.Zero()`（或在 C++ 中为：`YawMode::Zero()` ）。


#### lookahead 和 adaptive_lookahead

当您要求飞行器跟随路径时，AirSim 使用“胡萝卜跟随(carrot following)”算法。该算法通过向前观察路径并调整其速度矢量来运行。该算法的参数由 `lookahead` 和 `adaptive_lookahead` 指定。大多数情况下，您只需设置 `lookahead = -1` 和 `adaptive_lookahead = 0` 即可让算法自动确定值。


## 在真实载具上使用 API

我们希望能够在模拟环境中运行与真实车辆*相同的代码*。这样您就可以在模拟器中测试代码，并将其部署到真实载具上。

因此，一般来说，API 不应允许您执行在实车上无法执行的操作（例如，获取真实值）。当然，模拟器拥有更多信息，这对于可能不关心在实车上运行的应用程序非常有用。因此，我们通过添加 `sim` 前缀（例如 `simGetGroundTruthKinematics`）来明确区分仅用于模拟的 API。这样，如果您关心在实车上运行代码，就可以避免使用这些仅用于模拟的 API。

AirLib 是一个独立的库，您可以将其安装在机外计算模块（例如技嘉的 Mini PC 准系统）上。该模块可以使用完全相同的代码和飞行控制器协议与 PX4 等飞行控制器通信。您在模拟器中编写的测试代码保持不变。请参阅 [定制无人机上的 AirLib](custom_drone.md) 。


## 向 AirSim 添加新的 API

请参阅 [添加新 API](adding_new_apis.md)  页面

## References and Examples

* [C++ API 例子](apis_cpp.md)
* [Car 例子](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car)
* [多旋翼例子](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor)
* [计算机视觉例子](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision)
* [沿路径移动](https://github.com/Microsoft/AirSim/wiki/moveOnPath-demo) 演示展示了多旋翼飞行器快速穿越模块化街区的视频
* [Building a Hexacopter](https://github.com/Microsoft/AirSim/wiki/hexacopter)
* [Building Point Clouds](https://github.com/Microsoft/AirSim/wiki/Point-Clouds)


## FAQ

#### Unreal is slowed down dramatically when I run API
If you see Unreal getting slowed down dramatically when Unreal Engine window loses focus then go to 'Edit->Editor Preferences' in Unreal Editor, in the 'Search' box type 'CPU' and ensure that the 'Use Less CPU when in Background' is unchecked.

#### Do I need anything else on Windows?
You should install VS2019 with VC++, Windows SDK 10.0 and Python. To use Python APIs you will need Python 3.5 or later (install it using Anaconda).

#### Which version of Python should I use?
We recommend [Anaconda](https://www.anaconda.com/download/) to get Python tools and libraries. Our code is tested with Python 3.5.3 :: Anaconda 4.4.0. This is important because older version have been known to have [problems](https://stackoverflow.com/a/45934992/207661).

#### I get error on `import cv2`
You can install OpenCV using:
```
conda install opencv
pip install opencv-python
```

#### TypeError: unsupported operand type(s) for *: 'AsyncIOLoop' and 'float'

This error happens if you install Jupyter, which somehow breaks the msgpackrpc library.  Create a new python environment
which the minimal required packages.
