# ROS2 工作空间

工作空间可以简单理解为工程目录。

ROS 系统中一个典型的工作空间结构如图所示：

![image-20240511215226647](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240511215226647.png)

- **dev_ws：**根目录，里面会有四个子目录（子空间）；
- **build：**编译空间，里面是编译过程中产生的中间文件；
- **install：**安装空间，里面是编译得到的可执行文件和脚本；
- **log：**日志空间，里面是编译和运行过程中产生的各种警告、错误、信息等日志；
- **src：**代码空间，未来编写的代码、脚本，都需要人为的放置到这里。

## 创建工作空间

```shell
mkdir -p ~/dev_ws/src
```

然后在 `~/dev_ws/src` 目录下编写代码。

## 自动安装依赖

```shell
sudo apt install python3-pip -y
sudo pip install rosdepc
sudo rosdepc init
rosdepc update
cd ~/dev_ws
rosdepc install -i --from-path src --rosdistro humble -y
```

## 编译工作空间

```shell
sudo apt install python3-colcon-ros
cd ~/dev_ws
colcon build
```

编译成功后，就可以在工作空间 `dev_ws` 中看到自动生成的 `build`、`log`、`install` 文件夹了。

## 设置环境变量

```shell
source install/local_setup.sh # 仅在当前终端生效
echo " source ~/dev_ws/install/local_setup.sh" >> ~/.bashrc # 所有终端均生效
```

## 参考链接

- [https://book.guyuehome.com/ROS2/2.%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5/2.1_%E5%B7%A5%E4%BD%9C%E7%A9%BA%E9%97%B4/](https://book.guyuehome.com/ROS2/2.%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5/2.1_%E5%B7%A5%E4%BD%9C%E7%A9%BA%E9%97%B4/)

- [https://www.guyuehome.com/35408](https://www.guyuehome.com/35408)