2025/07/08

以下教程将会帮助你在Ubuntu20.04中配置一个ROS和ROS2共存的环境，以便于库的搬运和更新

# 0.版本问题
不同的Ubuntu各自有着对应的ROS版本，比如:
ver         ROS        ROS2
18.04     melodic  
20.04    noetic      foxy
22.04    \               humble
除了22.04后无法安装ROS以外，其余系统都可以安装ROS2，20.04前可以安装ROS，所以目前20.04是最高的双ROS共存版本
!!!预先用鱼香ros等下载好需要的ROS版本!!!
# 1.环境配置
以下是ROS(2)常用的环境：
```
ROS_DISTRO         #指明ROS当前版本(如foxy)
ROS_PACKAGE_PATH   #ROS的安装包路径
ROS_ROOT           #ROS的根路径
ROS_PYTHON_VERSION #ROS的Python版本
AMENT_CMAKE_PATH   #ROS2的Cmake路径
CATKIN_CMAKE_PATH  #ROS的Cmake路径
ROS_VERSION        #ROS的版本
```
其中
```
ROS_DISTRO
ROS_PACKAGE_PATH
```
需要在切换版本时清除
# 2.脚本参考
“一山不容二虎”
版本切换：
```
rosver(){
unset ROS_DISTRO
unset ROS_PACKAGE_PATH
case $1 in
        1)
                source  /opt/ros/noetic/setup.bash
                source ~/ROS工作空间/devel/setup.bash;;

        2)
                source  /opt/ros/foxy/setup.bash
                source ~/ROS2工作空间/install/setup.bash;;

        *)
                rosver 1;; #无输入默认ROS1
esac
}

#写完之后记得把原来的src删掉
```
“编译完之后要多敲代码...”
快速source .bashrc:
```
srcbashs(){
        rosv=${ROS_DISTRO}
        source ~/.bashrc
        if [ $rosv ];then
                if [ $rosv = foxy ];then rosver 2
                elif [ $rosv = noetic ];then rosver 1
                fi
        elif [[ $1 -eq 1 || $1 -eq 2 ]];then
                rosver $1
        fi
}

useage:
srcbashs     #如果原本就没有ROS环境，无操作
srcbashs 1/2 #载入ROS1/2的环境
srcbashs     #存在ROS环境时，保留原ROS环境继续source
```
“我怎么知道我现在在哪个环境里面?”
改变终端开头提示
```
export PS1="\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$" 
#PS1是你打开终端后前面那一串用户名的环境变量
#你可以随便在喜欢的地方加上标识(或者直接玩出花来)
#配合前面的rosver()来改变PS1从而随时看到当前环境
#最终成果：
name@user:(ROS Noetic)~$
```
# 3.验证
ROS验证：
```
#终端1
roscore
#终端2
rosrun turtlesim turtlesim_node
#终端3
rosrun turtlesim draw_square

rostopic echo/pub
```
ROS2验证：
```
#终端1
ros2 run turtlesim turtlesim_node
#终端2
ros2 run turtlesim draw_square

ros2 topic echo/pub
```
这里推荐双开，确保两个ROS是互相分离的，理论上，你应该能看到两个命名都是turtle1的乌龟在不同的窗口上面分别画方框
# 4.其他问题
### 1: rqt会开谁的？
在谁的环境下就会开谁的，不过两者的插件并不互通
(没错，你会看到ROS2的rqt没有那条could not find roscore了)
### 2: rviz会开谁的？
ROS：rviz
ROS2：rviz2
注意别敲错了
### 3: plotjuggler会读谁的？
ROS: rosrun plotjuggler plotjuggler
ROS2: ros2 run plotjuggler plotjuggler
记得先下载对应版本的plotjuggler(算是一种附属包)
### 4:  gazebo会用谁的？
ROS Noetic: 支持Gazebo 11.X
ROS 2 Foxy: 支持Gazebo Citadel
需要下载两个gazebo
### 5: #待补充 