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

您可以从 [releases](https://github.com/Microsoft/AirSim/releases) 获取 AirSim 二进制文件，也可以从源代码编译（[Windows](build_windows.md)、[Linux](build_linux.md)）。一旦您可以运行 AirSim，请选择 Car 作为载具，然后导航到 `PythonClient\car\` 文件夹并运行：

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

**笔记**
1. 您可能会注意到在我们的示例文件夹中有一个 `setup_path.py` 文件。这个文件有一个简单的代码来检测 `airsim` 包是否在父文件夹中可用，在这种情况下，我们使用它而不是 pip 安装的包，所以您总是使用最新的代码。
2. AirSim 仍在大量开发中，这意味着您可能需要经常更新软件包以使用新的 API。

## C++ 用户

如果要使用 C++ 的 API 和示例，请参阅 [C++ APIs 指南](apis_cpp.md) 。


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

## Hello Drone
下面介绍了如何使用 Python 的 AirSim API 来控制模拟四旋翼飞行器（另请参阅 [ C++ 示例](apis_cpp.md#hello_drone)）：

```python
# 准备运行示例：PythonClient/multirotor/hello_drone.py
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

You can also use following API call to set the sun position according to given date time:

```
simSetTimeOfDay(self, is_enabled, start_datetime = "", is_start_datetime_dst = False, celestial_clock_speed = 1, update_interval_secs = 60, move_sun = True)
```

The `is_enabled` parameter must be `True` to enable time of day effect. If it is `False` then sun position is reset to its original in the environment.

Other parameters are same as in [settings](settings.md#timeofday).

### Line-of-sight and world extent APIs
To test line-of-sight in the sim from a vehicle to a point or between two points, see simTestLineOfSightToPoint(point, vehicle_name) and simTestLineOfSightBetweenPoints(point1, point2), respectively.
Sim world extent, in the form of a vector of two GeoPoints, can be retrieved using simGetWorldExtents().

### Weather APIs
By default all weather effects are disabled. To enable weather effect, first call:

```
simEnableWeather(True)
```

Various weather effects can be enabled by using `simSetWeatherParameter` method which takes `WeatherParameter`, for example,

```
client.simSetWeatherParameter(airsim.WeatherParameter.Rain, 0.25);
```
The second parameter value is from 0 to 1. The first parameter provides following options:

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

Please note that `Roadwetness`, `RoadSnow` and `RoadLeaf` effects requires adding [materials](https://github.com/Microsoft/AirSim/tree/main/Unreal/Plugins/AirSim/Content/Weather/WeatherFX) to your scene.

Please see [example code](https://github.com/Microsoft/AirSim/blob/main/PythonClient/environment/weather.py) for more details.

### Recording APIs

Recording APIs can be used to start recording data through APIs. Data to be recorded can be specified using [settings](settings.md#recording). To start recording, use -

```
client.startRecording()
```

Similarly, to stop recording, use `client.stopRecording()`. To check whether Recording is running, call `client.isRecording()`, returns a `bool`.

This API works alongwith toggling Recording using R button, therefore if it's enabled using R key, `isRecording()` will return `True`, and recording can be stopped via API using `stopRecording()`. Similarly, recording started using API will be stopped if R key is pressed in Viewport. LogMessage will also appear in the top-left of the viewport if recording is started or stopped using API.

Note that this will only save the data as specfied in the settings. For full freedom in storing data such as certain sensor information, or in a different format or layout, use the other APIs to fetch the data and save as desired. Check out [Modifying Recording Data](modify_recording_data.md) for details on how to modify the kinematics data being recorded.

### Wind API

Wind can be changed during simulation using `simSetWind()`. Wind is specified in World frame, NED direction and m/s values

E.g. To set 20m/s wind in North (forward) direction -

```python
# Set wind to (20,0,0) in NED (forward direction)
wind = airsim.Vector3r(20, 0, 0)
client.simSetWind(wind)
```

Also see example script in [set_wind.py](https://github.com/Microsoft/AirSim/blob/main/PythonClient/multirotor/set_wind.py)

### Lidar APIs
AirSim offers API to retrieve point cloud data from Lidar sensors on vehicles. You can set the number of channels, points per second, horizontal and vertical FOV, etc parameters in [settings.json](settings.md).

More on [lidar APIs and settings](lidar.md) and [sensor settings](sensors.md)

### Light Control APIs

Lights that can be manipulated inside AirSim can be created via the `simSpawnObject()` API by passing either `PointLightBP` or `SpotLightBP` as the `asset_name` parameter and `True` as the `is_blueprint` parameter. Once a light has been spawned, it can be manipulated using the following API:

* `simSetLightIntensity`: This allows you to edit a light's intensity or brightness. It takes two parameters, `light_name`, the name of the light object returned by a previous call to `simSpawnObject()`, and `intensity`, a float value.

### Texture APIs

Textures can be dynamically set on objects via these APIs:

* `simSetObjectMaterial`: This sets an object's material using an existing Unreal material asset. It takes two string parameters, `object_name` and `material_name`.
* `simSetObjectMaterialFromTexture`: This sets an object's material using a path to a texture. It takes two string parameters, `object_name` and `texture_path`.

### Multiple Vehicles
AirSim supports multiple vehicles and control them through APIs. Please [Multiple Vehicles](multi_vehicle.md) doc.

### Coordinate System
All AirSim API uses NED coordinate system, i.e., +X is North, +Y is East and +Z is Down. All units are in SI system. Please note that this is different from coordinate system used internally by Unreal Engine. In Unreal Engine, +Z is up instead of down and length unit is in centimeters instead of meters. AirSim APIs takes care of the appropriate conversions. The starting point of the vehicle is always coordinates (0, 0, 0) in NED system. Thus when converting from Unreal coordinates to NED, we first subtract the starting offset and then scale by 100 for cm to m conversion. The vehicle is spawned in Unreal environment where the Player Start component is placed. There is a setting called `OriginGeopoint` in [settings.json](settings.md) which assigns geographic longitude, longitude and altitude to the Player Start component.

## Vehicle Specific APIs
### APIs for Car
Car has followings APIs available:

* `setCarControls`: This allows you to set throttle, steering, handbrake and auto or manual gear.
* `getCarState`: This retrieves the state information including speed, current gear and 6 kinematics quantities: position, orientation, linear and angular velocity, linear and angular acceleration. All quantities are in NED coordinate system, SI units in world frame except for angular velocity and accelerations which are in body frame.
* [Image APIs](image_apis.md).

### APIs for Multirotor
Multirotor can be controlled by specifying angles, velocity vector, destination position or some combination of these. There are corresponding `move*` APIs for this purpose. When doing position control, we need to use some path following algorithm. By default AirSim uses carrot following algorithm. This is often referred to as "high level control" because you just need to specify high level goal and the firmware takes care of the rest. Currently lowest level control available in AirSim is `moveByAngleThrottleAsync` API.

#### getMultirotorState
This API returns the state of the vehicle in one call. The state includes, collision, estimated kinematics (i.e. kinematics computed by fusing sensors), and timestamp (nano seconds since epoch). The kinematics here means 6 quantities: position, orientation, linear and angular velocity, linear and angular acceleration. Please note that simple_slight currently doesn't support state estimator which means estimated and ground truth kinematics values would be same for simple_flight. Estimated kinematics are however available for PX4 except for angular acceleration. All quantities are in NED coordinate system, SI units in world frame except for angular velocity and accelerations which are in body frame.

#### Async methods, duration and max_wait_seconds
Many API methods has parameters named `duration` or `max_wait_seconds` and they have *Async* as suffix, for example, `takeoffAsync`. These methods will return immediately after starting the task in AirSim so that your client code can do something else while that task is being executed. If you want to wait for this task to complete then you can call `waitOnLastTask` like this:

```cpp
//C++
client.takeoffAsync()->waitOnLastTask();
```

```cpp
# Python
client.takeoffAsync().join()
```

If you start another command then it automatically cancels the previous task and starts new command. This allows to use pattern where your coded continuously does the sensing, computes a new trajectory to follow and issues that path to vehicle in AirSim. Each newly issued trajectory cancels the previous trajectory allowing your code to continuously do the update as new sensor data arrives.

All *Async* method returns `concurrent.futures.Future` in Python (`std::future` in C++). Please note that these future classes currently do not allow to check status or cancel the task; they only allow to wait for task to complete. AirSim does provide API `cancelLastTask`, however.

#### drivetrain
There are two modes you can fly vehicle: `drivetrain` parameter is set to `airsim.DrivetrainType.ForwardOnly` or `airsim.DrivetrainType.MaxDegreeOfFreedom`. When you specify ForwardOnly, you are saying that vehicle's front should always point in the direction of travel. So if you want drone to take left turn then it would first rotate so front points to left. This mode is useful when you have only front camera and you are operating vehicle using FPV view. This is more or less like travelling in car where you always have front view. The MaxDegreeOfFreedom means you don't care where the front points to. So when you take left turn, you just start going left like crab. Quadrotors can go in any direction regardless of where front points to. The MaxDegreeOfFreedom enables this mode.

#### yaw_mode
`yaw_mode` is a struct `YawMode` with two fields, `yaw_or_rate` and `is_rate`. If `is_rate` field is True then `yaw_or_rate` field is interpreted as angular velocity in degrees/sec which means you want vehicle to rotate continuously around its axis at that angular velocity while moving. If `is_rate` is False then `yaw_or_rate` is interpreted as angle in degrees which means you want vehicle to rotate to specific angle (i.e. yaw) and keep that angle while moving.

You can probably see that when `yaw_mode.is_rate == true`, the `drivetrain` parameter shouldn't be set to `ForwardOnly` because you are contradicting by saying that keep front pointing ahead but also rotate continuously. However if you have `yaw_mode.is_rate = false` in `ForwardOnly` mode then you can do some funky stuff. For example, you can have drone do circles and have yaw_or_rate set to 90 so camera is always pointed to center ("super cool selfie mode"). In `MaxDegreeofFreedom` also you can get some funky stuff by setting `yaw_mode.is_rate = true` and say `yaw_mode.yaw_or_rate = 20`. This will cause drone to go in its path while rotating which may allow to do 360 scanning.

In most cases, you just don't want yaw to change which you can do by setting yaw rate of 0. The shorthand for this is `airsim.YawMode.Zero()` (or in C++: `YawMode::Zero()`).

#### lookahead and adaptive_lookahead
When you ask vehicle to follow a path, AirSim uses "carrot following" algorithm. This algorithm operates by looking ahead on path and adjusting its velocity vector. The parameters for this algorithm is specified by `lookahead` and `adaptive_lookahead`. For most of the time you want algorithm to auto-decide the values by simply setting `lookahead = -1` and `adaptive_lookahead = 0`.

## Using APIs on Real Vehicles
We want to be able to run *same code* that runs in simulation as on real vehicle. This allows you to test your code in simulator and deploy to real vehicle.

Generally speaking, APIs therefore shouldn't allow you to do something that cannot be done on real vehicle (for example, getting the ground truth). But, of course, simulator has much more information and it would be useful in applications that may not care about running things on real vehicle. For this reason, we clearly delineate between sim-only APIs by attaching `sim` prefix, for example, `simGetGroundTruthKinematics`. This way you can avoid using these simulation-only APIs if you care about running your code on real vehicles.

The AirLib is self-contained library that you can put on an offboard computing module such as the Gigabyte barebone Mini PC. This module then can talk to the flight controllers such as PX4 using exact same code and flight controller protocol. The code you write for testing in the simulator remains unchanged. See [AirLib on custom drones](custom_drone.md).

## 向 AirSim 添加新的 API

请参阅 [添加新 API](adding_new_apis.md)  页面

## References and Examples

* [C++ API Examples](apis_cpp.md)
* [Car Examples](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car)
* [Multirotor Examples](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor)
* [Computer Vision Examples](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision)
* [Move on Path](https://github.com/Microsoft/AirSim/wiki/moveOnPath-demo) demo showing video of fast multirotor flight through Modular Neighborhood environment
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
