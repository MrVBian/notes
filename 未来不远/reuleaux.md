# 1 容器

kinetic-u16.sh  

```sh
#!/bin/bash
XSOCK=/tmp/.X11-unix
xhost +local:root
docker run -it -d --name kinetic-u16 \
        -e DISPLAY=$DISPLAY -w /Projects \
        -v /home/zme/project/ros:/Projects -v $XSOCK:$XSOCK \
        -v $HOME/.Xauthority:/root/.Xauthority \
        -v /dev/bus/usb:/dev/bus/usb \
        --privileged --net=host \
        ros:kinetic-perception-xenial "$@"
```



# 2 环境

```shell
环境配置：
sudo apt-get install cmake g++ git ipython minizip python-dev python-h5py python-numpy python-scipy qt4-dev-tools
sudo apt-get install libassimp-dev libavcodec-dev libavformat-dev libavformat-dev libboost-all-dev libboost-date-time-dev libbullet-dev libfaac-dev libglew-dev libgsm1-dev liblapack-dev liblog4cxx-dev libmpfr-dev libode-dev libogg-dev libpcrecpp0v5 libpcre3-dev libqhull-dev libqt4-dev libsoqt-dev-common libsoqt4-dev libswscale-dev libswscale-dev libvorbis-dev libx264-dev libxml2-dev libxvidcore-dev

echo "\
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse">/etc/apt/sources.list
apt-get update
apt-get install vim -y

mkdir -p ~/.config/pip
echo "\
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
">~/.config/pip/pip.conf


echo "\
source /opt/ros/kinetic/setup.bash
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
" >> ~/.bashrc
source ~/.bashrc
```



```shell
git clone https://github.com/crigroup/openrave-installation
sudo ./install-dependencies.sh
sudo ./install-osg.sh
sudo ./install-fcl.sh
sudo ./install-openrave.sh

# 删除mpmath：
sudo apt remove python-mpmath

# X Error: BadAccess (attempt to access private resource denied
export QT_X11_NO_MITSHM=1

# 以上4个sh成功执行后，在终端输入下面命令测试。
openrave.py --example hanoi
```



# 3 问题处理

```shell
# Cannot find -lhdf5_hl and -lhdf5
# https://github.com/BVLC/caffe/issues/4333

# 其中urdf中的filename
export ROBOT=zmebot
export ROBOT=zmebot_dual_manipulator
rosrun xacro xacro.py $ROBOT.xacro > $ROBOT.urdf
rosrun collada_urdf urdf_to_collada $ROBOT.urdf $ROBOT.dae

# 设置精度，精度太高计算量过大，五位小数。
rosrun moveit_kinematics round_collada_numbers.py $ROBOT.dae $ROBOT.dae "5"

# 查看关节数据
openrave-robot.py "$ROBOT".dae --info links
# 3D显示
openrave "$ROBOT".dae

# 求解
# https://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/ikfast/ikfast_tutorial.html
# For a 6DOF arm:
python `openrave-config --python-dir`/openravepy/_openravepy_/ikfast.py --robot="$ROBOT".dae --iktype=transform6d --baselink="0" --eelink="8" --savefile=$(pwd)/"$ROBOT".cpp

# ur5
python `openrave-config --python-dir`/openravepy/_openravepy_/ikfast.py --robot=ur5.dae --iktype=transform6d --baselink=0 --eelink=9 --savefile=$(pwd)/ur5.cpp

# For a 7 dof arm, you will need to specify a free link:
python `openrave-config --python-dir`/openravepy/_openravepy_/ikfast.py --robot="$ROBOT".dae --iktype=transform6d --baselink="0" --eelink="8" --freeindex="1" --savefile=$(pwd)/"$ROBOT".cpp

# zme
python `openrave-config --python-dir`/openravepy/_openravepy_/ikfast.py --robot="$ROBOT".dae --iktype=transform6d --baselink="1" --eelink="8" --freeindex="1" --savefile=$(pwd)/"$ROBOT".cpp
```



```shell
# 使用ikfast61.cpp求逆解
cp /usr/local/lib/python2.7/dist-packages/openravepy/_openravepy_/ikfast.h .
g++ "$ROBOT".cpp -o "$ROBOT" -llapack -std=c++11


# - IKFast 算法依赖于LAPACK（Linear Algebra PACKag）库，所以编译的时候要链接 liblapack.so 库；
# - 输入为 3x4 矩阵（rotation 3x3， translation 3x1）
./"$ROBOT" 0.04071115 -0.99870914 0.03037599 0.4720009 -0.99874455 -0.04156303 -0.02796067 0.12648243 0.0291871 -0.02919955 -0.99914742 0.43451169
```


```c++
修改map_creator/include/map_creator/kinematics.h 头文件

#ifndef KINEMATICS
#define IKFAST_HAS_LIBRARY // Build IKFast with API functions
#define IKFAST_NO_MAIN

#define IK_VERSION 61
#include "mh5_ikfast.cpp"
#include <stdio.h>
#include <stdlib.h>
#include<vector>
#include<ros/ros.h>
#include <geometry_msgs/Pose.h>
```

第五行头文件换成编译后的cpp文件$ROBOT.cpp



```shell
# 设置机器人的运动学
cp "$ROBOT".cpp ./src/reuleaux/map_creator/include/map_creator/

# 重新编译
catkin_make
# 创建可达性地图
# 请不要试图将分辨率调得太低。安全门槛是0.05。否则，可能需要数小时，或者您的系统可能停止响应)。如果用户未提供任何分辨率，则默认分辨率为0.08
rosrun map_creator create_reachability_map 0.08 "$ROBOT".h5

# 可达性地图
rosrun map_creator load_reachability_map "$ROBOT".h5
```



# 4 使用



```shell
# Generate Ik Solver
python `openrave-config --python-dir`/openravepy/_openravepy_/ikfast.py --robot=<myrobot_name>.dae --iktype=transform6d --baselink=1 --eelink=8 --savefile=<ikfast_output_path>
# 使用ikfast61.cpp求逆解
cp /usr/local/lib/python2.7/dist-packages/openravepy/_openravepy_/ikfast.h .
g++ ikfast61.cpp -o ikfast -llapack -std=c++11


# - IKFast 算法依赖于LAPACK（Linear Algebra PACKag）库，所以编译的时候要链接 liblapack.so 库；
# - 输入为 3x4 矩阵（rotation 3x3， translation 3x1）
./ikfast 0.04071115 -0.99870914 0.03037599 0.4720009 -0.99874455 -0.04156303 -0.02796067 0.12648243 0.0291871 -0.02919955 -0.99914742 0.43451169

roscore
rosrun rviz rviz
# add_compile_options(-std=c++11)
catkin_make -DCMAKE_CXX_FLAGS="-std=c++11"


rosrun tf static_transform_publisher 0 0 0 0 0 0 1 world base_link 100
rosrun map_creator load_reachability_map ur5.h5

roscore
# Rviz下，添加ReachabilityMap后，topic：/reachability_map
rosrun rviz rviz

# apt-get install ros-kinetic-rqt-tf-tree
rosrun rqt_tf_tree rqt_tf_tree

./ikfast 0.04071115 -0.99870914 0.03037599 0.4720009 -0.99874455 -0.04156303 -0.02796067 0.12648243 0.0291871 -0.02919955 -0.99914742 0.43451169
```



# 5 参考

https://www.cswamp.com/post/131

http://wiki.ros.org/reuleaux

https://blog.csdn.net/Kalenee/article/details/80740258