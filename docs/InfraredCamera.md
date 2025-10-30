# 热红外图像

本教程将介绍如何使用 AirSim 和 AirSim Africa 环境生成模拟热红外 (infrared, IR) 图像。

预编译的非洲环境可以从此 Github 仓库的“发布”选项卡下载：
[Windows 预编译二进制包](https://github.com/OpenHUTB/air/releases)

要生成您自己的数据，您可以使用 2 个 python 文件： [create_ir_segmentation_map.py](https://github.com/OpenHUTB/air/tree/main/PythonClient//computer_vision/create_ir_segmentation_map.py) 和 
[capture_ir_segmentation.py](https://github.com/OpenHUTB/air/tree/main/PythonClient//computer_vision/capture_ir_segmentation.py).

[create_ir_segmentation_map.py](https://github.com/OpenHUTB/air/tree/main/PythonClient//computer_vision/create_ir_segmentation_map.py) 使用温度、发射率和相机响应信息来估算环境中物体的热数字计数，然后重新分配 AirSim 中的分割 ID 以匹配这些数字计数。它应该在开始采集热红外数据之前运行。否则，红外图像中的数字计数将不正确。相机响应、温度和发射率数据均已包含在内，适用于非洲环境。

[capture_ir_segmentation.py](https://github.com/OpenHUTB/air/tree/main/PythonClient//computer_vision/capture_ir_segmentation.py) 在分割 ID 重新分配后运行。它跟踪感兴趣的目标，并记录多旋翼飞行器拍摄的红外图像和场景图像。它使用计算机视觉(Computer Vision)模式。

最后，关于如何估算非洲环境中动植物温度等细节，请参阅这篇论文：

    @inproceedings{bondi2018airsim,
      title={AirSim-W: A Simulation Environment for Wildlife Conservation with UAVs},
      author={Bondi, Elizabeth and Dey, Debadeepta and Kapoor, Ashish and Piavis, Jim and Shah, Shital and Fang, Fei and Dilkina, Bistra and Hannaford, Robert and Iyer, Arvind and Joppa, Lucas and others},
      booktitle={Proceedings of the 1st ACM SIGCAS Conference on Computing and Sustainable Societies},
      pages={40},
      year={2018},
      organization={ACM}
    }

