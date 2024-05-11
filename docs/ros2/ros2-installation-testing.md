# ROS2 安装与测试

## ROS2 安装

基于 Ubuntu 22.04 LTS 操作系统。

### 1. 设置编码

```shell
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 
echo 'export LANG=en_US.UTF-8' >> ~/.bashrc
source ~/.bashrc
```

### 2. 添加源

```shell
sudo apt update && sudo apt install curl gnupg lsb-release 
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg 
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

> 如遇报错 “**Failed to connect to raw.githubusercontent.com**”，可参考
>
> https://www.guyuehome.com/37844

### 3. 安装 ROS2

```shell
sudo apt update
sudo apt upgrade
sudo apt install ros-humble-desktop
```

### 4. 设置环境变量

```shell
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc 
source ~/.bashrc
```

## ROS2 示例测试

测试底层通信系统 DDS 是否正常。

### 实例一：命令行实例

启动第一个终端，执行以下命令：

```shell
ros2 run demo_nodes_cpp talker
```

启动第二个终端，执行以下命令：

```shell
ros2 run demo_nodes_py listener
```

![image-20240511211721534](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240511211721534.png)

若 “Hello World” 能够在两个终端中正常传输，则说明底层通信系统 DDS 正常。

### 实例二：小海龟仿真实例

启动第一个终端，执行以下命令：

```shell
ros2 run turtlesim turtlesim_node
```

启动第二个终端，执行以下命令：

```shell
ros2 run turtlesim turtle_teleop_key
```

![image-20240511212041031](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240511212041031.png)

第一个终端启动了一个小海龟仿真器，第二个终端可以输入指令（上下左右）来控制小海龟运动。

## 参考链接

[https://book.guyuehome.com/ROS2/1.%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84/1.3_ROS2%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/](https://book.guyuehome.com/ROS2/1.%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84/1.3_ROS2%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/)

