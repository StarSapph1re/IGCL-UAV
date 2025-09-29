
# P450机上代码 环境配置步骤

在执行 `p450 compile.sh` 之前，需要完成以下依赖安装和配置步骤。

## 1. 安装 `multi_map_server`

```bash
sudo apt-get install ros-noetic-multi-map-server
sudo apt-get install ros-noetic-ddynamic-reconfigure
```

## 2. 安装 CUDA、cuDNN 和 TensorRT
参考nvidia驱动 550.144.03
请参考以下链接进行安装：
[安装 CUDA, cuDNN 和 TensorRT](https://blog.csdn.net/Apple_Coco/article/details/129293019)

Unbuntu20.04中，建议cuda选择run文件安装，cudnn选择直接下载archive，tensorrt下载tar包（不建议采用deb安装文件，会有诸多问题）
TensorRT配置bashrc：
```
export TENSORRT_DIR=/usr/local/TensorRT-8.6.1.6
export PATH=${TENSORRT_DIR}/bin:$PATH
export LD_LIBRARY_PATH=${TENSORRT_DIR}/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${TENSORRT_DIR}/lib:$LIBRARY_PATH
export CUDNN_DIR=/usr/local/cuda
```

## 3. 安装 OpenCV 4.7.0

安装 OpenCV 4.7.0 时，务必确保同时安装 `opencv_contrib`，否则会出现如下错误：  
`#include <opencv2/aruco.hpp>` 出错。

参考文章：[安装 OpenCV 4.7.0](https://blog.csdn.net/wxyczhyza/article/details/128968849)  
安装 `opencv_contrib` 参考链接：[腾讯云文章](https://cloud.tencent.com/developer/article/1491605)

### 配置 OpenCV 和 contrib 模块

编译 OpenCV 时，需要使用以下 `cmake` 配置命令：

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_GENERATE_PKGCONFIG=ON \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D INSTALL_C_EXAMPLES=ON \
      -D OPENCV_EXTRA_MODULES_PATH=/path/to/opencv_contrib/modules \
      -D OPENCV_EXAMPLES=ON ..
```

## 4. 编译时遇到 `float32_t` 未定义的问题

在编译过程中，如果遇到 `float32_t` 未定义的错误，定位到文件 `p450_experiment/src/spirecv-ros/sv-rosapp/samples/gimbal_server.cpp`，直接将 `float32_t` 替换为 `float`，目前没有发现其他问题。

## 5. 安装 RealSense SDK

克隆 RealSense SDK 仓库并进行编译安装：

```bash
git clone -b v2.55.1 https://github.com/IntelRealSense/librealsense --recursive
cd librealsense
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/home/<usrname>  # 必须安装到自己用户名对应的目录下
make
sudo make install
```

安装完成后，修改 `realsense2Config.cmake` 配置文件：

```bash
# 修改文件 /<path-to>/librealsense/build/realsense2Config.cmake
# 将第 35 行更改为：
include("/home/<usrname>/lib/cmake/realsense2/realsense2Targets.cmake")
```

如果还是报错找不到librealsense，终端中输入：
```
export CMAKE_PREFIX_PATH=/path/to/realsense:$CMAKE_PREFIX_PATH
```

## 6. 安装 Miniconda

```bash
mkdir -p ~/build_env/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/build_env/miniconda3/miniconda.sh
bash ~/build_env/miniconda3/miniconda.sh -b -u -p ~/build_env/miniconda3
rm ~/build_env/miniconda3/miniconda.sh
```

## 7. 解决 `catkin_make / cmake` 找不到 OpenCV 路径问题

在 `catkin_make` 或 `cmake` 时，如果出现找不到 OpenCV 路径的错误，可以通过以下命令指定 OpenCV 路径：

```bash
export OpenCV_DIR=/path/to/opencv-4.7.0/build
```

注意：不要同时在 WSL 中使用 `sudo apt-get` 安装的 OpenCV 和自己编译的 OpenCV，可能会导致头文件定义冲突。若出现此问题，卸载 `sudo apt-get` 安装的 OpenCV：

```bash
sudo apt-get remove libopencv-*
```

卸载之后因为依赖问题会把ros的一些包也卸载可以，重新用小鱼的安装器进行安装。

如果出现/usr/bin/ld: 找不到 -lnvinfer，

## 8. 安装`cv_bridge`

如果需要在 Python 中使用 `cv_bridge`，可以按照以下步骤进行安装：

```bash
git clone -b noetic https://github.com/ros-perception/vision_opencv.git --recursive
cd vision_opencv/cv_bridge
pip setup.py install
```

## 可能遇到的其他问题

如果在编译带有 Python 脚本的功能包时，遇到以下错误：

```
AttributeError: module 'em' has no attribute 'RAW_OPT'
```

可以通过安装指定版本的 `empy` 来解决：

```bash
pip install empy==3.3.4
```

