# AirSim 中的多辆载具

自 1.2 版本起，AirSim 已全面支持多载具模式。此功能允许您轻松创建多架载具，并使用 API 来控制它们。


## 创建多辆载具

只需在 [settings.json](settings.md) 文件中指定即可。`Vehicles` 元素允许您指定要创建的车辆列表，以及它们的初始位置和方向。位置以 NED 坐标系指定，单位为 SI，原点位于虚幻引擎环境中的玩家起始组件。方向以偏航角 (Yaw)、俯仰角 (Pitch) 和横滚角 (Roll) 表示，单位为度。


### 创建多辆车

```json
{
	"SettingsVersion": 1.2,
	"SimMode": "Car",

	"Vehicles": {
		"Car1": {
		  "VehicleType": "PhysXCar",
		  "X": 4, "Y": 0, "Z": -2
		},
		"Car2": {
		  "VehicleType": "PhysXCar",
		  "X": -4, "Y": 0, "Z": -2,
      "Yaw": 90
		}
  }
}
```

### 创建多个无人机

```json
{
	"SettingsVersion": 1.2,
	"SimMode": "Multirotor",

	"Vehicles": {
		"Drone1": {
		  "VehicleType": "SimpleFlight",
		  "X": 4, "Y": 0, "Z": -2,
      "Yaw": -180
		},
		"Drone2": {
		  "VehicleType": "SimpleFlight",
		  "X": 8, "Y": 0, "Z": -2
		}

    }
}
```

## 使用 API 管理多辆车

自 AirSim 1.2 版本起，新的 API 允许您指定 `vehicle_name`。此名称对应于 json 设置中的键（例如，上面的 Car1 或 Drone2）。


[多旋翼飞行器示例代码](https://github.com/OpenHUTB/air/blob/main/PythonClient/multirotor/multi_agent_drone.py)

[汽车示例代码](https://github.com/OpenHUTB/air/blob/main/PythonClient/car/multi_agent_car.py)

使用 API 管理多辆车需要指定 `vehicle_name`，该名称必须硬编码到脚本中，或者需要解析设置文件。此外，还有一个简单的 API `listVehicles()`，它返回一个包含当前车辆名称的字符串列表（在 C++ 中为 vector）。例如，对于上述 2 辆车的设置 -

```python
>>> client.listVehicles()
['Car1', 'Car2']
```

### 演示

[![AirSimMultiple Vehicles Demo Video](images/demo_multi_vehicles.png)](https://youtu.be/35dgcuLuF5M)

### 通过 API 在运行时创建载具

在 AirSim 的最新主分支中，可以使用 `simAddVehicle` API 在运行时创建载具。这便于创建大量载具，而无需在设置中指定它们。目前此功能存在一些限制，如下所述 -


`simAddVehicle` 函数接受以下参数：

- `vehicle_name`: 要创建的载具名称，此名称对于每载具（包括在 settings.json 中定义的任何现有车辆）都应该是唯一的。 
- `vehicle_type`: 载具类型，例如“simpleflight”。目前仅支持 SimpleFlight、PhysXCar 和 ComputerVision，且仅在其各自的模拟模式下支持。其他载具类型，包括 PX4 和 ArduPilot 相关类型，均不受支持。 
- `pose`: 载具初始姿态
- `pawn_path`: 载具蓝图路径，默认为空，它使用载具类型的默认蓝图  Vehicle blueprint path, default empty wbich uses the default blueprint for the vehicle type

返回：`bool` 载具是否已创建

创建载具后，可以使用常规 API 通过 `vehicle_name` 参数来控制载具并与之交互。目前尚无法指定其他设置，例如额外的摄像头等，未来可能会增加传递载具设置的 JSON 字符串的功能。此外，它还支持上述的 `listVehicles()` API，因此生成的载具将被添加到列表中。


例如，请查看 [HelloSpawnedDrones.cpp](https://github.com/OpenHUTB/air/blob/main/HelloSpawnedDrones/HelloSpawnedDrones.cpp) -

![HelloSpawnedDrones](images/HelloSpawnedDrones.gif)

以及 [runtime_car.py](https://github.com/OpenHUTB/air/tree/main/PythonClient/car/runtime_car.py) -

![runtime_car](images/simAddVehicle_Car.gif)
