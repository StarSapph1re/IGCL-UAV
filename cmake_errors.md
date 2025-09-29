### building通过 linking报大量undefined reference ros::spin() 等错误
CMakeLists.txt要把target_link_libraries补充完整，参考其他文件的写法

### undefined symbol: _ZN6google21kLogSiteUninitializedE
glog相关，catkin_make前务必conda deactivate，如果已经编译，在build文件夹中找到对应的功能包删除后不在conda下编译即可

### 编译ceres-solver
```
mkdir build
mkdir install
cd build/
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_STANDARD=14 -DCMAKE_CXX_FLAGS="-march=native" -DCMAKE_INSTALL_PREFIX="../install/" -DBUILD_EXAMPLES=OFF
make -j4 install
```

### anaconda库路径污染
问题现象：链接动态库的时候提示 undefined reference to "libxxx"  
但是apt install发现已经安装这个库  
将/anaconda3/lib中相关的libxxx.so重命名为backup也无法解决  
尝试et(CMAKE_IGNORE_PATH "/home/hanyx/usr/anaconda3/lib;/home/hanyx/usr/anaconda3/include")无果  
命令catkin_make VERBOSE=1 |& tee build.log可以将链接过程保存到log，会发现里面始终有anaconda的路径  
下面的脚本是【从一个完全“空白”的环境启动，不继承当前 shell 的任何变量】，能有效防止anaconda路径污染
```
#!/usr/bin/env bash
set -euo pipefail

# === 1) 定义完全干净的环境（不继承你现在的 shell 变量） ===
env -i \
HOME="$HOME" USER="$USER" SHELL=/bin/bash \
# 只允许系统工具在 PATH 中
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
# 强制使用系统 pkg-config
PKG_CONFIG="/usr/bin/pkg-config" \
PKG_CONFIG_PATH="/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/share/pkgconfig" \
# 链接/运行时仅搜系统库与 ROS
LD_LIBRARY_PATH="/usr/lib/x86_64-linux-gnu:/usr/local/lib:/opt/ros/noetic/lib" \
# CMake 仅在这些前缀下找包
CMAKE_PREFIX_PATH="/usr:/usr/local:/opt/ros/noetic" \
# 显式给出系统头/库（有些 FindXXX 用得到）
CMAKE_LIBRARY_PATH="/usr/lib/x86_64-linux-gnu:/usr/local/lib" \
CMAKE_INCLUDE_PATH="/usr/include:/usr/local/include" \
bash -lc '
  set -euo pipefail
  echo "[INFO] Using pkg-config: $(which pkg-config)"
  echo "[INFO] pkg-config pc_path: $(pkg-config --variable pc_path pkg-config)"

  cd ~/usr/vins

  # === 2) 彻底清缓存，避免旧配置残留 ===
  rm -rf build devel

  # === 3) 运行 catkin_make（加一些保险旗标）===
  catkin_make \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_EXE_LINKER_FLAGS="-Wl,--no-as-needed" \
    -DCMAKE_IGNORE_PATH="/home/$USER/usr/anaconda3/lib;/home/$USER/usr/anaconda3/include" \
    -DCMAKE_SYSTEM_PREFIX_PATH="/usr;/usr/local;/opt/ros/noetic" \
    -DBoost_NO_BOOST_CMAKE=ON

  echo "[OK] Build finished."
'

```