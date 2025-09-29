# Prometheus仿真 环境配置步骤

## 1. 编译 Prometheus

首先需要编译 Prometheus，按照官网 [Prometheus 仿真环境配置](https://wiki.amovlab.com/public/prometheus-wiki/%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B/Prometheus%E4%BB%BF%E7%9C%9F%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE_Ubuntu/prometheus_px4%E9%85%8D%E7%BD%AE.html) 进行配置。

## 2. 编译通信模块

在编译通信模块时，可能会遇到以下问题：

### 错误：`No module named 'catkin_pkg'`
解决方案：

```bash
pip install catkin_pkg
```

### 错误：`/usr/include/boost/thread/pthread/thread_data.hpp:60:5: error: missing binary operator before token "("`
解决方案：

找到 `thread_data.hpp` 文件，定位到以下内容：

```cpp
#if PTHREAD_STACK_MIN>0
```

将其替换为：

```cpp
#ifdef PTHREAD_STACK_MIN
```

## 3. 编译 Gazebo 仿真模块

### 错误：`No rule to make target '/usr/lib/x86_64-linux-gnu/libpthread.so'`
解决方案：

```bash
sudo apt-get install libpthread-stubs0-dev
```

检查 `/usr/lib/x86_64-linux-gnu/` 目录，确保存在 `libpthread.so.0`，如果缺少符号链接，可以手动创建：

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libpthread.so.0 /usr/lib/x86_64-linux-gnu/libpthread.so
```

### 错误：`No rule to make target '/usr/lib/x86_64-linux-gnu/libdl.so'`
解决方案：

检查 `/usr/lib/x86_64-linux-gnu/` 目录，确保存在 `libdl.so.2`，如果缺少符号链接，可以手动创建：

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libdl.so.2 /usr/lib/x86_64-linux-gnu/libdl.so
```

## 4. 编译 Fast-lio 模块

### 错误：`fatal error: fast_lio/Pose6D.h: No such file or directory`
解决方案：

此问题是由于没有添加依赖，导致 C++ 编译时找不到消息文件。修改 `CMakeLists.txt` 文件：

在 `add_executable` 行下添加：

```cmake
add_dependencies(fastlio_mapping ${PROJECT_NAME}_generate_messages_cpp)
```

## 5. 编译目标检测模块

### 安装 SpireCV 和流媒体服务器

首先需要安装 SpireCV，并在 SpireCV 主目录下运行 `scripts/x86-intel/zlm-server-install.sh` 来安装流媒体服务器。只有在安装完 ZLM 后，才能实现图像推流。

### 启动流媒体服务器

安装完 ZLM 后，运行 `SpireCV/ZLM/MediaServer` 启动流媒体服务器。默认端口为 `554`。由于权限问题，需要使用 `sudo` 来启动 MediaServer，或者可以配置命令不需要 `sudo`：

1. 执行 `sudo visudo`
2. 添加以下行：

```bash
<username> ALL=(ALL) NOPASSWD: /home/<username>/usr/SpireCV/ZLM/MediaServer
```

> 注意：`MediaServer` 会加载 `ZLM/` 目录下的配置文件。虽然配置文件可以写在其他路径，但是修改 `config` 文件后才会生效。

## 6. 仿真时可能遇到的错误

### 错误：`resource not found px4`

此问题与 `~/.bashrc` 中 `source` 命令的顺序有关。`source devel/setup.bash` 会覆盖之前的 PX4 设置。

### 解决方案：

1. 检查当前环境里是否正确配置了 PX4 环境：

```bash
rospack list | grep "px4"
```

如果没有输出：

2. 确保 `px4` 的 `source` 在 `bashrc` 的最后：

```bash
source ~/px4/ROM/setup.bash
```

3. 在运行 `sh` 文件之前，执行：

```bash
source ~/.bashrc
```

## 7. 下载 Prometheus 地面站（非必须）

参考 [Prometheus 地面站准备工作](https://wiki.amovlab.com/public/prometheus-wiki/Prometheus%E5%9C%B0%E9%9D%A2%E7%AB%99/%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C.html) 来下载和设置地面站。


