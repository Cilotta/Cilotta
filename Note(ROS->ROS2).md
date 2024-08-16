```
本记中使用的符号含义：
>>> 表示原本就有但是更新了的句段
+++ 表示新加入的句段
--- 表示删除的句段，以及一些不应该再出现在文件中的元素
```
#TODO
# 0. What it changes?
* 1. 不再需要开 ros core ，ros2 是一种去中心化的系统
  tips: 没有 master 了(X)，master被整合到一起不需要特意开了(O)
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
	   (--- devel)
``` 
# 1. CMakeLists.txt
```
#文件结构（大致版）
cmake_minimum_required(VERSION 3.5)
set (CMAKE_CXX_STANDARD 17)
find_package(ament_cmake REQUIRED)
find_package(xxx_package REQUIRED)

include_directories(include)
add_library(xxx xxx)
add_executable(xxx xxx)

ament_target_dependencies(xxx xxx)
#或者可以沿用ros的 target_link_libraries(xxx xxx)

install(xxx) #相应接口顺着下面的文档部分改

ament_export_libraries(xxx)
ament_export_dependencies(xxx)

ament_package()

```
cmake的最低版本要求为 3.5
C++ 版本要求 C++14 以上
新加了三个选项，与ROS2的编译有关
包的接口及使用改变
ROS2 允许使用 set( ) 来快速设置参数
信息的生成和输出需要添加
```
+++ rosidl_generate_interfaces (***)
+++ ament_register_extention(***)
>>> ament_export_dependencies(***)
```
# 2.核心构件 Node,Topic,Service,Action 的变化
## Node
和 ROS 中相比，有关的命令变化不大，如 node list 等仍然可以使用
```
#ROS
ros::init(argc,argv,"*node name*");
ros::NodeHandle n;
ros::Publisher xxx_pub = n.advertise<x_msg::xxx>(xxx,1000);
ros::Rate loop_rate(10);

#ROS2
rclcpp::init(argc,argv);
auto node = rclcpp::Node::make_shared(*node name*);
auto xxx_pub = node->create_publisher<x_msg::msg::xxx>(xxx,1000);
rclcpp::Rate loop_rate(10);

#Time
ros::Time -> rclcpp::Time
std_msgs::Time -> builtin_interface::msg::Time

#TODO :
话说回来，builtin_interface::msg::Time time和builtin_interface__msg__Time time，这两个time到底有什么区别啊?
```
此外，新增加了一个叫做 LifecycyleNode 的类，允许你控制节点的起停，初始化，ROS1的节点启动是无序且会出现依赖冲突的，ROS2则可以解决这个问题
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
#### *Hints: 当你在编译时遇到以下报错：
```
#1 KeyError processing template 'struct.idl.em': 'time'  Call Stack (most recent call first):   

#2 NameError: '***' is an invalid field name.It should have the
  pattern 'xxx'
```
#1: 使用你的 IDE，搜索 “time”，在 ros2 中，时间戳被整合到了 builtin_interfaces/Time，因此，原 ROS 中所有使用了时间戳 time 的地方都要修改为 builtin_interfaces/Time  (!大小写区分!)
#2: ROS2 的命名规则现在更严格了，如 bool 值中不能有大写等，用 IDE 搜索并按照报错的 pattern 替换掉相应的命名即可
# 3.*Change in Interfaces.*
*catin 已经不复存在，自然，你也不应该再在文件中看到它*
以下是部分接口的转移：
- `CATKIN_DEPENDS` 和 `DEPENDS` 的参数被传递给新的函数 [ament_export_dependencies](https://github.com/ament/ament_cmake/blob/master/ament_cmake_export_dependencies/cmake/ament_export_dependencies.cmake) 
- `CATKIN_GLOBAL_BIN_DESTINATION`: `bin` 
- `CATKIN_GLOBAL_INCLUDE_DESTINATION`: `include`    
- `CATKIN_GLOBAL_LIB_DESTINATION`: `lib`
- `CATKIN_GLOBAL_LIBEXEC_DESTINATION`: `lib`
- `CATKIN_GLOBAL_SHARE_DESTINATION`: `share`
- `CATKIN_PACKAGE_BIN_DESTINATION`: `lib/${PROJECT_NAME}`
- `CATKIN_PACKAGE_INCLUDE_DESTINATION`: `include/${PROJECT_NAME}` 
- `CATKIN_PACKAGE_LIB_DESTINATION`: `lib`
- `CATKIN_PACKAGE_SHARE_DESTINATION`: `share/${PROJECT_NAME}`
*[Check this !](http://dev.ros2.fishros.com/doc/Contributing/Migration-Guide.html)*

# 4.Things should TODO in the cpp & hpp
#1: msg调用改变, ==Before:== geometry::xxx ; ==Now:== geometry::msg::xxx ,此外，#include尽量使用 .hpp 文件，而不是 .h

#2: XmlRpc 是 ROS 的节点通讯方式，其用于辅助节点间建立连接，并由 Master 发布，而 ROS2 选择了 DDS 的通讯方式 ，显而易见，XmlRpc在 ROS2 这个无 Master 的系统之中是无法使用的\

# #TODO :
## 1: 所有文件内的XmlRpc类需要替代
暂时不知道怎么改
## 2：rm2_referee内的serial类需要替代
暂时不知道怎么改
## 3: ROS_WARN等数据流需要修改
改成RCLCPP_WARN,，但是接口变了需要更多参数，目前暂时不知道怎么改