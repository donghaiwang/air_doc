# XBox 控制器

要将 XBox 控制器与 AirSim 一起使用，请按照以下步骤操作：

1. 连接 XBox 控制器，以便它显示在您的 PC 游戏控制器中：

![Gamecontrollers](images/game_controllers.png)

2. 启动 QGroundControl，您应该会在设置下看到一个新的操纵杆选项卡：

![Gamecontrollers](images/qgc_joystick.png)

现在校准无线电，并设置一些方便的按钮操作。例如，我的设置是`A`键启动无人机，`B`键将其置于手动飞行模式，`X`键将其置于高度保持模式，`Y`键将其置于位置保持模式。我还更喜欢勾选“在横滚、俯仰和偏航时使用指数曲线”复选框时控制器的感觉，因为这能让我更好地感知细微的移动。

QGroundControl 将通过上述 MavLinkTest 设置的 UDP 代理端口 14550 找到您的 Pixhawk。AirSim 将通过上述 MavLinkTest 设置的另一个 UDP 服务器端口 14570 找到您的 Pixhawk。此时，您也可以使用所有 QGroundControl 控件进行自主飞行。


3. 使用 MavLinkTest.exe 连接到 Pixhawk 串行端口，如下所示：
```
MavLinkTest.exe -serial:*,115200 -proxy:127.0.0.1:14550 -server:127.0.0.1:14570
```

4. 使用以下 `~/Documents/AirSim/settings.json` 设置运行 AirSim 虚幻模拟器：
```
"Vehicles": {
    "PX4": {
        "VehicleType": "PX4Multirotor",

        "SitlIp": "",
        "SitlPort": 14560,
        "UdpIp": "127.0.0.1",
        "UdpPort": 14570,
        "UseSerial": false
    }
}
```

## 高级

如果 QGroundControl 中未显示“操纵杆”选项卡，请点击工具栏左侧的紫色`Q`图标，打开“首选项”面板。前往“常规”选项卡，勾选“虚拟操纵杆”复选框。返回设置屏幕（齿轮图标），点击“参数”选项卡，在搜索框中输入 `COM_RC_IN_MODE`，并将其值更改为 `操纵杆/无 RC 检查(Joystick/No RC Checks)` 或 `Virtual RC by Joystick(通过操纵杆进行虚拟 RC)`。


### 其他选项

查看 [遥控器选项](remote_control.md)