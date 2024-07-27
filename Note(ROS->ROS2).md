```
本记中使用的符号含义：
>>> 表示原本就有但是更新了的句段
+++ 表示新加入的句段
```
# 0. What it changes?
* 1. 不再需要开 ros core ，ros2 是一种去中心化的系统
* 2. 编译工具再次升级，从 catkin_tools 变成 colcon 
```
sudo apt install python3-colcon-common-extensions
### 编译后文件夹结构为：
.
└── workspace
    ├── build
    ├── install
    ├── log
    └── src
	   (也就是说没有 devel 了)
``` 
# 1. CMakeLists.txt
```
>>> cmake_minimum_required(VERSION 3.5)
>>> set (CMAKE_CXX_STANDARD 17)


+++ set (CMAKE_EXE_LINKER_FLAGS ***)
+++ set (CMAKE_SHARED_LINKER_FLAGS ***)
+++ set (LIBRARY_OUTPUT_PATH ***)

ROS                 ROS2
catkin_make       | ament_make
roscpp            | rclcpp
                  |
add_message_files | set(msg_files ***)
```
cmake的最低版本要求为 3.5
C++ 版本要求 C++14 以上
新加了三个选项，与ROS2的编译有关
部分包改变
ROS2现在使用 set( ) 来设置参数
信息的生成和输出需要添加
```
+++ rosidl_generate_interfaces (***)
+++ ament_register_extention(***)
>>> ament_export_dependencies(***)
```
一些优化
