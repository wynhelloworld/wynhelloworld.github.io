# ROS2 功能包

在 ROS2 工作空间的 src 目录下进行编写的文件并不是普通的文件，而是被称作功能包。

## 创建功能包

**Usage:**

```shell
ros2 pkg create --build-type <build-type> <package_name>
```

**Description:**

- **pkg：**表示功能包相关的功能；
- **create：**表示创建功能包；
- **--build-type：**表示创建的功能包是用来写 C++ 的，还是写 Python 的：
  - 写 C++，则后面跟 **ament_cmake**；
  - 写 Python，则后面跟 **ament_python**。
- **package_name：**表示功能包的名字。

**Example：**

分别在终端中创建 C++ 和 Python 版本的功能包：

```shell
cd ~/dev_ws/src
ros2 pkg create --build-type ament_cmake learning_pkg_c       # C++
ros2 pkg create --build-type ament_python learning_pkg_python # Python
```

## 编译功能包

```shell
cd ~/dev_ws
colcon build   # 编译工作空间的所有功能包
```

## 设置环境变量

为了能够让系统识别到编译出的程序，所以要配置环境变量：

```shell
source install/local_setup.sh # 仅在当前终端生效
echo " source ~/dev_ws/install/local_setup.sh" >> ~/.bashrc # 所有终端均生效
```

## 功能包的结构

### C++ 功能包结构

![image-20240512091124559](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240512091124559.png)

### Python 功能包结构

![image-20240512091148820](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240512091148820.png)

## 参考链接

[https://book.guyuehome.com/ROS2/2.%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5/2.2_%E5%8A%9F%E8%83%BD%E5%8C%85/](https://book.guyuehome.com/ROS2/2.%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5/2.2_%E5%8A%9F%E8%83%BD%E5%8C%85/)