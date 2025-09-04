# 使用 C++ APIs for AirSim

如果您还没有阅读 [通用 API 文档](apis.md) ，请先阅读。本文档描述了 C++ 示例和其他 C++ 相关的细节。


## 快速入门

最快的入门方法是在 Visual Studio 2019 中打开 AirSim.sln。您将在解决方案中看到 [Hello Car](https://github.com/Microsoft/AirSim/tree/main/HelloCar/) and [Hello Drone](https://github.com/Microsoft/AirSim/tree/main/HelloDrone/) 和 [Hello Drone](https://github.com/Microsoft/AirSim/tree/main/HelloDrone/) 的示例。这些示例将向您展示需要在 VC++ 项目中设置的包含路径和库路径。如果您使用的是 Linux，则可以在 cmake 文件或编译器命令行中指定这些路径。


#### 头文件和库的目录

* 头文件目录: `$(ProjectDir)..\AirLib\deps\rpclib\include;include;$(ProjectDir)..\AirLib\deps\eigen3;$(ProjectDir)..\AirLib\include`
* 依赖: `rpc.lib`
* 库文件目录: `$(ProjectDir)\..\AirLib\deps\MavLinkCom\lib\$(Platform)\$(Configuration);$(ProjectDir)\..\AirLib\deps\rpclib\lib\$(Platform)\$(Configuration);$(ProjectDir)\..\AirLib\lib\$(Platform)\$(Configuration)`
* 引用：将 AirLib 和 MavLinkCom 引用到项目引用中。（右键单击您的项目，然后前往`References`，`Add reference...`，然后选择 AirLib 和 MavLinkCom）

## Hello Car

以下是如何使用 C++ 的 AirSim API 来控制模拟汽车（另请参阅 [Python 示例](apis.md#hello_car) ）：

```cpp

// ready to run example: https://github.com/Microsoft/AirSim/blob/main/HelloCar/main.cpp

#include <iostream>
#include "vehicles/car/api/CarRpcLibClient.hpp"

int main()
{
    msr::airlib::CarRpcLibClient client;
    client.enableApiControl(true); //this disables manual control
    CarControllerBase::CarControls controls;

    std::cout << "Press enter to drive forward" << std::endl; std::cin.get();
    controls.throttle = 1;
    client.setCarControls(controls);

    std::cout << "Press Enter to activate handbrake" << std::endl; std::cin.get();
    controls.handbrake = true;
    client.setCarControls(controls);

    std::cout << "Press Enter to take turn and drive backward" << std::endl; std::cin.get();
    controls.handbrake = false;
    controls.throttle = -1;
    controls.steering = 1;
    client.setCarControls(controls);

    std::cout << "Press Enter to stop" << std::endl; std::cin.get();
    client.setCarControls(CarControllerBase::CarControls());

    return 0;
}
```

## Hello Drone

下面介绍了如何使用 C++ 的 AirSim API 来控制模拟四旋翼飞行器（另请参阅 [Python 示例](apis.md#hello_drone) ）：

```cpp

// ready to run example: https://github.com/Microsoft/AirSim/blob/main/HelloDrone/main.cpp

#include <iostream>
#include "vehicles/multirotor/api/MultirotorRpcLibClient.hpp"

int main()
{
    msr::airlib::MultirotorRpcLibClient client;

    std::cout << "Press Enter to enable API control\n"; std::cin.get();
    client.enableApiControl(true);

    std::cout << "Press Enter to arm the drone\n"; std::cin.get();
    client.armDisarm(true);

    std::cout << "Press Enter to takeoff\n"; std::cin.get();
    client.takeoffAsync(5)->waitOnLastTask();

    std::cout << "Press Enter to move 5 meters in x direction with 1 m/s velocity\n"; std::cin.get();
    auto position = client.getMultirotorState().getPosition(); // from current location
    client.moveToPositionAsync(position.x() + 5, position.y(), position.z(), 1)->waitOnLastTask();

    std::cout << "Press Enter to land\n"; std::cin.get();
    client.landAsync()->waitOnLastTask();

    return 0;
}
```

## 参见

* 如何在其它项目中使用 AirSim 内部基础设施的 [示例](https://github.com/microsoft/AirSim/tree/main/Examples) 
* [DroneShell](https://github.com/microsoft/AirSim/tree/main/DroneShell) 应用程序展示了如何使用 C++ API 制作简单的界面来控制无人机 
* [HelloSpawnedDrones](https://github.com/microsoft/AirSim/tree/main/HelloSpawnedDrones) 应用程序演示如何即时制造更多飞行器
* [Python APIs](apis.md)
