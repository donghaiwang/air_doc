# 图像 API

如果您不熟悉 AirSim API，请先阅读 [通用 API 文档](apis.md) 。

## 获取单张图像

以下示例代码演示如何从名为“0”的摄像头获取单张图像。返回值是 PNG 格式图像的字节数据。要获取未压缩图像和其他格式，以及了解可用的摄像头，请参阅后续章节。

### Python

```python
import airsim # pip install hutb

# 用于汽车的 CarClient()
client = airsim.MultirotorClient()

png_image = client.simGetImage("0", airsim.ImageType.Scene)
# 对图像进行一些操作
```

### C++

```cpp
#include "vehicles/multirotor/api/MultirotorRpcLibClient.hpp"

int getOneImage() 
{
    using namespace msr::airlib;
    
    // 用于汽车的 CarRpcLibClient
    MultirotorRpcLibClient client;

    std::vector<uint8_t> png_image = client.simGetImage("0", VehicleCameraBase::ImageType::Scene);
    // 对图像进行一些操作
}
```

## 以更加灵活的方式获取图像

`simGetImages` API 的使用比 `simGetImage` API 略微复杂一些。例如，您可以通过一次 API 调用获取左侧摄像头视图、右侧摄像头视图以及左侧摄像头的深度图像。`simGetImages` API 还允许您获取未压缩的图像以及单通道浮点图像（而不是 3 通道 (RGB)，每个通道 8 位）。


### Python

```python
import airsim  # pip install hutb

# 用于汽车的 CarClient()
client = airsim.MultirotorClient()

responses = client.simGetImages([
    # png 格式
    airsim.ImageRequest(0, airsim.ImageType.Scene), 
    # 未压缩的 RGB 数组字节
    airsim.ImageRequest(1, airsim.ImageType.Scene, False, False),
    # 浮点未压缩图像
    airsim.ImageRequest(1, airsim.ImageType.DepthPlanar, True)])
 
 # 对包含图像数据、姿态、时间戳等信息的响应进行处理
```

#### 使用 NumPy 处理 AirSim 图像

如果您计划使用 numpy 进行图像处理，您应该获取未压缩的 RGB 图像，然后像这样将其转换为 numpy 格式：

```python
responses = client.simGetImages([airsim.ImageRequest("0", airsim.ImageType.Scene, False, False)])
response = responses[0]

# 获取 NumPy 数组
img1d = np.fromstring(response.image_data_uint8, dtype=np.uint8) 

# 将数组重塑为 4 通道图像数组（高 x 宽 x 4）。
img_rgb = img1d.reshape(response.height, response.width, 3)

# 原图已垂直翻转
img_rgb = np.flipud(img_rgb)

# 写入 png
airsim.write_png(os.path.normpath(filename + '.png'), img_rgb) 
```

#### 快速提示
- API `simGetImage` 返回二进制`字符串字面量`，这意味着您可以直接将其导出为二进制文件以创建 .png 文件。但是，如果您想以其他方式处理它，可以使用便捷的函数 `airsim.string_to_uint8_array`。该函数将二进制字符串字面量转换为 NumPy uint8 数组。 

- API `simGetImages` 可以在一次调用中接受来自任何摄像头的多种图像类型的请求。您可以指定图像是 PNG 压缩图像、RGB 未压缩图像还是浮点数组。对于 PNG 压缩图像，您将获得一个`二进制字符串字面量`。对于浮点数组，您将获得一个 Python float64 列表。您可以使用以下方法将此浮点数组转换为 NumPy 二维数组： 
    ```
    airsim.list_to_2d_float_array(response.image_data_float, response.width, response.height)
    ```
    您还可以使用 `airsim.write_pfm()` 函数将浮点数组保存到 .pfm 文件（便携式浮点映射格式）。 

- 如果您希望在调用图像 API 时同步查询位置和方向信息，可以使用 `client.simPause(True)` 和 `client.simPause(False)` 在调用图像 API 和查询所需物理状态时暂停模拟，以确保在调用图像 API 后物理状态立即保持不变。 

### C++

```cpp
int getStereoAndDepthImages() 
{
    using namespace msr::airlib;
    
    typedef VehicleCameraBase::ImageRequest ImageRequest;
    typedef VehicleCameraBase::ImageResponse ImageResponse;
    typedef VehicleCameraBase::ImageType ImageType;

    // 用于汽车
    // CarRpcLibClient client;
    MultirotorRpcLibClient client;

    // 获取右图、左图和深度图。前两张图为 PNG 格式，后两张图为 float16 格式。
    std::vector<ImageRequest> request = { 
        //png 格式
        ImageRequest("0", ImageType::Scene),
        // 未压缩的 RGB 数组字节
        ImageRequest("1", ImageType::Scene, false, false),       
        // 浮点未压缩图像
        ImageRequest("1", ImageType::DepthPlanar, true) 
    };

    const std::vector<ImageResponse>& response = client.simGetImages(request);
    // 对包含图像数据、姿态、时间戳等信息的响应进行处理
}
```

## 可直接运行的完整示例

### Python

如需更完整的可直接运行的示例代码，请参阅 AirSimClient 项目中的多旋翼飞行器 [示例代码](https://github.com/Microsoft/AirSim/tree/main/PythonClient//multirotor/hello_drone.py) 或 [HelloCar 示例代码](https://github.com/Microsoft/AirSim/tree/main/PythonClient//car/hello_car.py) 。这些代码还演示了一些简单的操作，例如将图像保存到文件或使用 `numpy` 处理图像。


### C++

如需更完整的可运行 [示例代码](https://github.com/Microsoft/AirSim/tree/main/HelloDrone//main.cpp) ，请参阅 HelloDrone 多旋翼飞行器项目或 [HelloCar 项目的示例代码](https://github.com/Microsoft/AirSim/tree/main/HelloCar//main.cpp) 。


See also [other example code](https://github.com/Microsoft/AirSim/tree/main/Examples/DataCollection/StereoImageGenerator.hpp) that generates specified number of stereo images along with ground truth depth and disparity and saving it to [pfm format](pfm.md).

## Available Cameras

These are the default cameras already available in each vehicle. Apart from these, you can add more cameras to the vehicles and external cameras which are not attached to any vehicle through the [settings](settings.md).

### Car
The cameras on car can be accessed by following names in API calls: `front_center`, `front_right`, `front_left`, `fpv` and `back_center`. Here FPV camera is driver's head position in the car.
### Multirotor
The cameras on the drone can be accessed by following names in API calls: `front_center`, `front_right`, `front_left`, `bottom_center` and `back_center`. 
### Computer Vision Mode
Camera names are same as in multirotor.

### Backward compatibility for camera names
Before AirSim v1.2, cameras were accessed using ID numbers instead of names. For backward compatibility you can still use following ID numbers for above camera names in same order as above: `"0"`, `"1"`, `"2"`, `"3"`, `"4"`. In addition, camera name `""` is also available to access the default camera which is generally the camera `"0"`.

## "计算机视觉" 模式

您可以在所谓的“计算机视觉”模式下使用 AirSim。在此模式下，物理引擎将被禁用，并且没有飞行器，只有摄像头（如果您想要飞行器但不包含其运动学特性，可以使用多旋翼模式和物理引擎 [ExternalPhysicsEngine](settings.md##physicsenginename) ）。您可以使用键盘移动（使用 F1 查看按键帮助）。您可以按下录制按钮来连续生成图像。或者，您可以调用 API 来移动摄像头并拍摄图像。


要激活此模式，请编辑您可以在 Documents\AirSim 文件夹（或 Linux 上的 ~/Documents/AirSim）中找到的 [settings.json](settings.md)，并确保根级别存在以下值：

```json
{
  "SettingsVersion": 1.2,
  "SimMode": "ComputerVision"
}
```

以下是移动摄像头并拍摄图像的 [Python 代码示例](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/cv_mode.py) 。

此模式受到 [UnrealCV 项目](http://unrealcv.org/) 的启发。


### Setting Pose in Computer Vision Mode
To move around the environment using APIs you can use `simSetVehiclePose` API. This API takes position and orientation and sets that on the invisible vehicle where the front-center camera is located. All rest of the cameras move along keeping the relative position. If you don't want to change position (or orientation) then just set components of position (or orientation) to floating point nan values. The `simGetVehiclePose` allows to retrieve the current pose. You can also use `simGetGroundTruthKinematics` to get the quantities kinematics quantities for the movement. Many other non-vehicle specific APIs are also available such as segmentation APIs, collision APIs and camera APIs.

## Camera APIs
The `simGetCameraInfo` returns the pose (in world frame, NED coordinates, SI units) and FOV (in degrees) for the specified camera. Please see [example usage](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/cv_mode.py).

The `simSetCameraPose` sets the pose for the specified camera while taking an input pose as a combination of relative position and a quaternion in NED frame. The handy `airsim.to_quaternion()` function allows to convert pitch, roll, yaw to quaternion. For example, to set camera-0 to 15-degree pitch while maintaining the same position, you can use:
```
camera_pose = airsim.Pose(airsim.Vector3r(0, 0, 0), airsim.to_quaternion(0.261799, 0, 0))  #PRY in radians
client.simSetCameraPose(0, camera_pose);
```

- `simSetCameraFov` allows changing the Field-of-View of the camera at runtime.
- `simSetDistortionParams`, `simGetDistortionParams` allow setting and fetching the distortion parameters K1, K2, K3, P1, P2

All Camera APIs take in 3 common parameters apart from the API-specific ones, `camera_name`(str), `vehicle_name`(str) and `external`(bool, to indicate [External Camera](settings.md#external-cameras)). Camera and vehicle name is used to get the specific camera, if `external` is set to `true`, then the vehicle name is ignored. Also see [external_camera.py](https://github.com/microsoft/AirSim/blob/main/PythonClient/computer_vision/external_camera.py) for example usage of these APIs.

### Gimbal
You can set stabilization for pitch, roll or yaw for any camera [using settings](settings.md#gimbal).

Please see [example usage](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/cv_mode.py).

## Changing Resolution and Camera Parameters
To change resolution, FOV etc, you can use [settings.json](settings.md). For example, below addition in settings.json sets parameters for scene capture and uses "Computer Vision" mode described above. If you omit any setting then below default values will be used. For more information see [settings doc](settings.md). If you are using stereo camera, currently the distance between left and right is fixed at 25 cm.

```json
{
  "SettingsVersion": 1.2,
  "CameraDefaults": {
      "CaptureSettings": [
        {
          "ImageType": 0,
          "Width": 256,
          "Height": 144,
          "FOV_Degrees": 90,
          "AutoExposureSpeed": 100,
          "MotionBlurAmount": 0
        }
    ]
  },
  "SimMode": "ComputerVision"
}
```

## What Does Pixel Values Mean in Different Image Types?
### Available ImageType Values
```cpp
  Scene = 0, 
  DepthPlanar = 1, 
  DepthPerspective = 2,
  DepthVis = 3, 
  DisparityNormalized = 4,
  Segmentation = 5,
  SurfaceNormals = 6,
  Infrared = 7,
  OpticalFlow = 8,
  OpticalFlowVis = 9
```                

### DepthPlanar and DepthPerspective
You normally want to retrieve the depth image as float (i.e. set `pixels_as_float = true`) and specify `ImageType = DepthPlanar` or `ImageType = DepthPerspective` in `ImageRequest`. For `ImageType = DepthPlanar`, you get depth in camera plane, i.e., all points that are plane-parallel to the camera have same depth. For `ImageType = DepthPerspective`, you get depth from camera using a projection ray that hits that pixel. Depending on your use case, planner depth or perspective depth may be the ground truth image that you want. For example, you may be able to feed perspective depth to ROS package such as `depth_image_proc` to generate a point cloud. Or planner depth may be more compatible with estimated depth image generated by stereo algorithms such as SGM.

### DepthVis
When you specify `ImageType = DepthVis` in `ImageRequest`, you get an image that helps depth visualization. In this case, each pixel value is interpolated from black to white depending on depth in camera plane in meters. The pixels with pure white means depth of 100m or more while pure black means depth of 0 meters.

### DisparityNormalized
You normally want to retrieve disparity image as float (i.e. set `pixels_as_float = true` and specify `ImageType = DisparityNormalized` in `ImageRequest`) in which case each pixel is `(Xl - Xr)/Xmax`, which is thereby normalized to values between 0 to 1.

### Segmentation
When you specify `ImageType = Segmentation` in `ImageRequest`, you get an image that gives you ground truth segmentation of the scene. At the startup, AirSim assigns value 0 to 255 to each mesh available in environment. This value is then mapped to a specific color in [the pallet](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Content/HUDAssets/seg_color_palette.png). The RGB values for each object ID can be found in [this file](seg_rgbs.txt).

You can assign a specific value (limited to the range 0-255) to a specific mesh using APIs. For example, below Python code sets the object ID for the mesh called "Ground" to 20 in Blocks environment and hence changes its color in Segmentation view:

```python
success = client.simSetSegmentationObjectID("Ground", 20);
```

The return value is a boolean type that lets you know if the mesh was found.

Notice that typical Unreal environments, like Blocks, usually have many other meshes that comprises of same object, for example, "Ground_2", "Ground_3" and so on. As it is tedious to set object ID for all of these meshes, AirSim also supports regular expressions. For example, the code below sets all meshes which have names starting with "ground" (ignoring case) to 21 with just one line:

```python
success = client.simSetSegmentationObjectID("ground[\w]*", 21, True);
```

The return value is true if at least one mesh was found using regular expression matching.

It is recommended that you request uncompressed image using this API to ensure you get precise RGB values for segmentation image:
```python
responses = client.simGetImages([ImageRequest(0, AirSimImageType.Segmentation, False, False)])
img1d = np.fromstring(response.image_data_uint8, dtype=np.uint8) #get numpy array
img_rgb = img1d.reshape(response.height, response.width, 3) #reshape array to 3 channel image array H X W X 3
img_rgb = np.flipud(img_rgb) #original image is fliped vertically

#find unique colors
print(np.unique(img_rgb[:,:,0], return_counts=True)) #red
print(np.unique(img_rgb[:,:,1], return_counts=True)) #green
print(np.unique(img_rgb[:,:,2], return_counts=True)) #blue  
```

A complete ready-to-run example can be found in [segmentation.py](https://github.com/Microsoft/AirSim/tree/main/PythonClient//computer_vision/segmentation.py).

#### Unsetting object ID
An object's ID can be set to -1 to make it not show up on the segmentation image.

#### How to Find Mesh Names?
To get desired ground truth segmentation you will need to know the names of the meshes in your Unreal environment. To do this, you will need to open up Unreal Environment in Unreal Editor and then inspect the names of the meshes you are interested in using the World Outliner. For example, below we see the mesh names for the ground in Blocks environment in right panel in the editor:

![record screenshot](images/unreal_editor_blocks.png)

If you don't know how to open Unreal Environment in Unreal Editor then try following the guide for [building from source](build_windows.md).

Once you decide on the meshes you are interested, note down their names and use above API to set their object IDs. There are [few settings](settings.md#segmentation-settings) available to change object ID generation behavior.

#### Changing Colors for Object IDs
At present the color for each object ID is fixed as in [this pallet](https://github.com/microsoft/AirSim/blob/main/Unreal/Plugins/AirSim/Content/HUDAssets/seg_color_palette.png). We will be adding ability to change colors for object IDs to desired values shortly. In the meantime you can open the segmentation image in your favorite image editor and get the RGB values you are interested in.

#### Startup Object IDs
At the start, AirSim assigns object ID to each object found in environment of type `UStaticMeshComponent` or `ALandscapeProxy`. It then either uses mesh name or owner name (depending on settings), lower cases it, removes any chars below ASCII 97 to remove numbers and some punctuations, sums int value of all chars and modulo 255 to generate the object ID. In other words, all object with same alphabet chars would get same object ID. This heuristic is simple and effective for many Unreal environments but may not be what you want. In that case, please use above APIs to change object IDs to your desired values. There are [few settings](settings.md#segmentation-settings) available to change this behavior.

#### Getting Object ID for Mesh
The `simGetSegmentationObjectID` API allows you get object ID for given mesh name.

### Infrared
Currently this is just a map from object ID to grey scale 0-255. So any mesh with object ID 42 shows up with color (42, 42, 42). Please see [segmentation section](#segmentation) for more details on how to set object IDs. Typically noise setting can be applied for this image type to get slightly more realistic effect. We are still working on adding other infrared artifacts and any contributions are welcome.

### OpticalFlow and OpticalFlowVis
These image types return information about motion perceived by the point of view of the camera. OpticalFlow returns a 2-channel image where the channels correspond to vx and vy respectively. OpticalFlowVis is similar to OpticalFlow but converts flow data to RGB for a more 'visual' output.

## Example Code
A complete example of setting vehicle positions at random locations and orientations and then taking images can be found in [GenerateImageGenerator.hpp](https://github.com/Microsoft/AirSim/tree/main/Examples/DataCollection/StereoImageGenerator.hpp). This example generates specified number of stereo images and ground truth disparity image and saving it to [pfm format](pfm.md).
