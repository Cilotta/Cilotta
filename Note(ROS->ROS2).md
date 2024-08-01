```
本记中使用的符号含义：
>>> 表示原本就有但是更新了的句段
+++ 表示新加入的句段
--- 表示删除的句段，以及一些不应该再出现在文件中的元素
```
# 0. What it changes?
* 1. 不再需要开 ros core ，ros2 是一种去中心化的系统
  tips: 没有 master 了(X)，master被整合到一起不需要特意开了(O)
* 2. 编译工具再次升级，从 ```catkin_tools``` 变成 ```colcon```
	```
	用法：
	colcon build
	```
* 3. 编译后的结构改变了
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
# 2.核心构件 Node,Topic,Service,Action 的变化
## Node
和 ROS 中相比，有关的命令变化不大，如 node list 等仍然可以使用
## Topic
同 ROS，Topic 可以实现一对一至多对多通讯
==\*.msg 文件编写==
```
msg文件本身并无太大改变，编写方式同ROS
CMakeList.txt中：
	>>> find_package(rosidl_default_generators REQUIRED)
	+++ rosidl_default_generators(${PROJECT_NAME}
		## 相关msg文件(需要完整路径，如 msg/xxx.msg )
		)
	--- add_message_files()
		generate_massages()
		catkin* ,message_generation 不应该再出现

	>>> add_excuteable(#node_name )
package.xml中：
	>>> <build_depend>rosidl_default_generator</build_depend>
	>>> <exec_depend>rosidl_default_runtime</exec_depend>
	+++ <member_of_group>rosidl_interface_packages</member_of_group> 

```
## Service
和 ROS 不同的是，ROS2 中的服务只能有一个 Service Server ，但可以有多个 Service Client ，除此之外，参数服务器也被整合为参数文件，而无须特意加载
==\*.srv 文件编写==
```
srv文件本身并无太大改变，编写方式同ROS
CMakeList.txt中：
	+++ rosidl_default_generators(${PROJECT_NAME}
		## 相关srv文件(需要完整路径，如 srv/xxx.srv )
		)
	其余同msg编写方式
package.xml中：
	同msg编写方式
```
## Action
两者区别不大
==\*.action 文件编写==
```
action文件本身并无太大改变，编写方式同ROS
CMakeList.txt中：
	+++ rosidl_default_generators(${PROJECT_NAME}
		## 相关action文件(需要完整路径，如 action/xxx.action)
		)
	--- add_action_files()
		actionlib,actionlib_msgs 不应该再出现
	其余同msg编写方式
package.xml中：
	>>> <buildtool_depend>rosidl_default_generator</buildtool_depend>
	>>> <depend>action_msgs</depend>
	其余同msg编写方式 
```