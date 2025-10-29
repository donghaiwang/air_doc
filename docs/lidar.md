# 如何在 AirSim 中使用激光雷达

AirSim支持多旋翼飞行器和汽车的激光雷达。

激光雷达的启用及其他激光雷达设置可通过 AirSimSettings.json 文件进行配置。有关通用/共享传感器设置的配置信息，请参阅 [通用传感器](sensors.md) 部分。


## 在载具上启用激光雷达
* 默认情况下，激光雷达未启用。要启用激光雷达，请在设置 JSON 文件中设置 SensorType 和 Enabled 属性。 

```json
    "Lidar1": {
         "SensorType": 6,
         "Enabled" : true,
    }
```

* 一辆载具上可以安装多个激光雷达。

## 激光雷达配置

以下参数现在可以通过 settings.json 文件进行配置。

参数                 | 描述
--------------------------| ------------
NumberOfChannels          | 激光雷达的通道/激光器数量
Range                     | 范围，单位为米
PointsPerSecond           | 每秒捕获的点数
RotationsPerSecond        | 每秒旋转次数
HorizontalFOVStart        | 激光雷达水平视场角起始值（以度为单位）
HorizontalFOVEnd          | 激光雷达水平视场角（FOV）末端，单位为度
VerticalFOVUpper          | 激光雷达垂直视场角上限（以度为单位）
VerticalFOVLower          | 激光雷达垂直视场角下限（以度为单位）
X Y Z                     | 激光雷达相对于载具的位置（NED，单位：米）
Roll Pitch Yaw            | 激光雷达相对于载具的方位（以度为单位，偏航-俯仰-横滚顺序，以前方矢量 +X 为参考）
DataFrame                 | 输出点的坐标系（“车辆惯性坐标系(VehicleInertialFrame)”或“传感器本地坐标系(SensorLocalFrame)”）
ExternalController        | 是否将数据发送到外部控制器（例如 ArduPilot 或 PX4，如果使用）（默认为 `true`）（PX4 目前不发送激光雷达数据）

例如

```json
{
    "SeeDocsAt": "https://microsoft.github.io/AirSim/settings/",
    "SettingsVersion": 1.2,

    "SimMode": "Multirotor",

     "Vehicles": {
		"Drone1": {
			"VehicleType": "simpleflight",
			"AutoCreate": true,
			"Sensors": {
			    "LidarSensor1": {
					"SensorType": 6,
					"Enabled" : true,
					"NumberOfChannels": 16,
					"RotationsPerSecond": 10,
					"PointsPerSecond": 100000,
					"X": 0, "Y": 0, "Z": -1,
					"Roll": 0, "Pitch": 0, "Yaw" : 0,
					"VerticalFOVUpper": -15,
					"VerticalFOVLower": -25,
					"HorizontalFOVStart": -20,
					"HorizontalFOVEnd": 20,
					"DrawDebugPoints": true,
					"DataFrame": "SensorLocalFrame"
				},
				"LidarSensor2": {
				   "SensorType": 6,
					"Enabled" : true,
					"NumberOfChannels": 4,
					"RotationsPerSecond": 10,
					"PointsPerSecond": 10000,
					"X": 0, "Y": 0, "Z": -1,
					"Roll": 0, "Pitch": 0, "Yaw" : 0,
					"VerticalFOVUpper": -15,
					"VerticalFOVLower": -25,
					"DrawDebugPoints": true,
					"DataFrame": "SensorLocalFrame"
				}
			}
		}
    }
}
```

## 服务器端可视化调试

默认情况下，激光雷达点不会绘制在视口中。要启用在视口中绘制激光命中点，请通过 settings.json 文件启用 `DrawDebugPoints` 设置。

```json
    "Lidar1": {
         ...
         "DrawDebugPoints": true
    },
```

**注意：** 启用 `DrawDebugPoints` 可能会导致 `v1.3.1` 和 `v1.3.0` 版本中内存占用过高并导致程序崩溃。此问题已在主分支中修复，应该会在后续版本中得到解决。

## 客户端 API

使用 `getLidarData()` API 获取激光雷达数据。

* API 返回一个点云，它是浮点数的扁平数组，同时还包含捕获时间戳和激光雷达姿态。
* Point-Cloud:
    * 浮点数表示上次扫描中该范围内每个命中点的 [x,y,z] 坐标。
    * 输出数据点的框架可以通过“DataFrame”属性进行配置 -
        * "" 或 `VehicleInertialFrame` -- 默认; 返回的点位于载具惯性坐标系中（NED 格式，单位为米）。
        * `SensorLocalFrame` -- 返回的点坐标系为激光雷达局部坐标系（NED格式，单位为米）。
* Lidar Pose:
    * 载具惯性系中的激光雷达姿态（单位：米，单位：NED）
    * 可用于将点转换到其他坐标系。
* Segmentation: 对每个激光雷达点碰撞对象进行分割 

### Python 例子
- [drone_lidar.py](https://github.com/OpenHUTB/air/blob/main/PythonClient/multirotor/drone_lidar.py)
- [car_lidar.py](https://github.com/OpenHUTB/air/blob/main/PythonClient/car/car_lidar.py)
- [sensorframe_lidar_pointcloud.py](https://github.com/OpenHUTB/air/blob/main/PythonClient/multirotor/sensorframe_lidar_pointcloud.py)
- [vehicleframe_lidar_pointcloud.py](https://github.com/OpenHUTB/air/blob/main/PythonClient/multirotor/vehicleframe_lidar_pointcloud.py)

## 即将推出
* 在客户端可视化激光雷达数据。
