# 经验/问题记录
## - 让ros识别到添加的python结点
对应所在功能包的CMakeList.txt加上
```
catkin_install_python(PROGRAMS src/UVLTrack/uvl_track_node.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```
## - 编译时需要用到特定python环境（如conda）

编译时加上
```
-DPYTHON_EXECUTABLE=/for/example/miniconda3/envs/uvl/bin/python
```
同时python结点内第一行也要手动指定
```
#!/for/example/miniconda3/envs/uvl/bin/python
```

## - 解决 `ERROR: package name 'uvl-track-ros' is illegal and cannot be used in message generation` 错误

在 ROS 中，包名只能包含小写字母、数字和下划线。如果遇到类似错误，可以通过修改 `CMakeLists.txt` 文件中的 `project` 项，以及 `package.xml` 文件中的 `<name>` 元素来更改包名。

## - 有关相机参数读取
p450自带的相机参数.yaml文件与pyyaml读取方式不兼容，需要手动复制一份，去掉header以及!!opencv-matrix

## - QGC地面站报错 GLIBC_2.35 not found
https://blog.csdn.net/dou3516/article/details/144448667


## - 话题消息checksum报错
如果用仿真的workspace启动结点和真机代码workspace启动结点进行通讯，可能出现二者话题的数据类型相同，但是消息内容（msg文件）不同的情况，即使msg文件仅有细微的差别，在两边分别编译也会出现checksum不同，会出现如下错误：
```
Client [/yolov5_tracking] wants topic /uav1/prometheus/state to have datatype/md5sum [prometheus_msgs/UAVState/7fa47b335536d2bbef37443391737bef], but our version has [prometheus_msgs/UAVState/19d12342d45fd9dbd79a3c5ff86556ac]. Dropping connection
```
需要找到两边的这个话题的消息数据类型对应的msg文件，保证内容完全一致，然后把devel, build删了全部重新编译（有点耗时，但只编译消息部分不行，如何找到指定的文件删除重编译有待探索）

## - 查看某个端口是否被占用
```
sudo lsof -i :1935
```
