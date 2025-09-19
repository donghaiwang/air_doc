# 升级 API 客户端代码

AirSim v1.2中有几个API更改，我们希望消除不一致性，增加未来的扩展性，并提供更干净的接口。然而，其中许多更改都是*破坏性的更改*，这意味着您需要修改与AirSim对话的客户端代码。


## 更快的方法

虽然您需要在客户端代码中做的大多数更改都相当容易，但更快捷的方法就是查看示例代码（例如 [Hello Drone](https://github.com/OpenHUTB/air/tree/main/PythonClient/multirotor/hello_drone.py) 或 [Hello Car](https://github.com/OpenHUTB/air/tree/main/PythonClient/car/hello_car.py) ）以了解更改的要点。

## 导入 AirSim
不是

```python
from AirSimClient import *
```
而是使用这个：

```python
import airsim
```

以上假设您已经使用以下方式安装了 AirSim 模块，
```
pip install --user airsim
```

如果您正在从 repo 中的 PythonClient 文件夹运行代码，那么您也可以执行以下操作：

```python
import setup_path 
import airsim
```

这里 setup_path.py 应该存在于你的文件夹中，它将设置 `airsim` 包在 `PythonClient` repo 文件夹中的路径。PythonClient 文件夹中的所有示例都使用此方法。

## 使用 AirSim 类

由于我们现在已将所有内容都包含在包中，因此您需要为 AirSim 类使用明确的命名空间，如下所示。

不是

```python
client1 = CarClient()
```

而是使用这个：

```python
client1 = airsim.CarClient()
```

## AirSim 类型

我们已将所有类型移至 `airsim` 命名空间。

不是

```python
image_type = AirSimImageType.DepthVis

d = DrivetrainType.MaxDegreeOfFreedom
```

而是使用这个：

```python
image_type = airsim.ImageType.DepthVis

d = airsim.DrivetrainType.MaxDegreeOfFreedom
```

## 获取图像

以下内容并无新意，只是上述内容的组合。请注意，所有之前使用 `camera_id` 的 API 现在均改为使用 `camera_name`。您可以点击此处查看 [可用的相机](image_apis.md#avilable_cameras) 。


不是

```python
responses = client.simGetImages([ImageRequest(0, AirSimImageType.DepthVis)])
```

而是使用这个：

```python
responses = client.simGetImages([airsim.ImageRequest("0", airsim.ImageType.DepthVis)])
```

## 实用方法

在早期版本中，我们提供了一些实用方法作为 `AirSimClientBase` 的一部分。现在，这些方法已移至 `airsim` 命名空间，以提供更具 Python 风格的接口。


不是

```python
AirSimClientBase.write_png(my_path, img_rgba) 

AirSimClientBase.wait_key('Press any key')
```

而是使用这个：

```python
airsim.write_png(my_path, img_rgba)

airsim.wait_key('Press any key')
```

## 相机名称

AirSim 现在使用 [名称](image_apis.md#available_cameras) （而非索引号）来引用摄像头。但为了保持向后兼容性，这些名称将使用旧索引号作为字符串别名。

不是

```python
client.simGetCameraInfo(0)
```

而是使用这个：

```python
client.simGetCameraInfo("0")

# 或者

client.simGetCameraInfo("front-center")
```

## 异步方法

对于多旋翼飞行器，AirSim 中存在多种方法，例如 `takeoff` 或 `moveByVelocityZ`，这些方法需要较长时间才能完成。现在，所有这些方法都已重命名，并添加了后缀 *Async*，如下所示。

不是

```python
client.takeoff()

client.moveToPosition(-10, 10, -10, 5)
```

而是使用这个：

```python
client.takeoffAsync().join()

client.moveToPositionAsync(-10, 10, -10, 5).join()
```

这里 `.join()` 是对 Python `Future` 类的调用，用于等待异步调用完成。你也可以选择在调用过程中执行其他计算。

## 纯模拟方法

现在，我们已经清楚地区分了仅在模拟中可用的方法和可能在实际车辆上可用的方法。仅用于模拟的方法以 `sim` 为前缀，如下所示。

```
getCollisionInfo()      重命名为       simGetCollisionInfo()
getCameraInfo()         重命名为       simGetCameraInfo()
setCameraOrientation()  重命名为       simSetCameraOrientation()
```

## 状态信息

此前，`CarState` 仅包含模拟信息，例如 `kinematics_true`。未来，`CarState` 将仅包含可在现实世界中获取的信息。


```python
k = car_state.kinematics_true
```

使用这个：

```python
k = car_state.kinematics_estimated

# 或者

k = client.simGetGroundTruthKinematics()
```