# ROS2 节点

机器人的每一项功能，都被称为是一个节点。

- 每个节点都是一个独立的可执行程序；
- 每个节点都可以用不同的编程语言来编写、编译得到；
- 一个机器人的众多节点可能分布在不同的计算机上（分布式）；
- 每个节点都需要一个唯一的命名。

## 示例：创建并运行成功一个节点

### 1. 创建功能包

```shell
cd ~/dev_ws/src
ros2 pkg create --build-type ament_cmake learning_node
```

### 2. 编写源文件、CMakeLists.txt、package.xml

learning_node/src/node_helloworld.cpp

```c++
#include <unistd.h>
#include "rclcpp/rclcpp.hpp"

/**
 * 创建一个 HelloWorld 节点, 初始化时输出 “hello world” 日志
 */
class HelloWorldNode : public rclcpp::Node
{
public:
    HelloWorldNode()
        : Node("node_helloworld_class")                      
    {
        while(rclcpp::ok())                                  
        {
            RCLCPP_INFO(this->get_logger(), "Hello World");  
            sleep(1);                                        
        }
    }
};

// ROS2 节点主入口 main 函数
int main(int argc, char *argv[])                               
{

    // ROS2 C++ 接口初始化
    rclcpp::init(argc, argv);        
    
    // 创建 ROS2 节点对象并进行初始化                 
    rclcpp::spin(std::make_shared<HelloWorldNode>()); 
    
    // 关闭 ROS2 C++ 接口
    rclcpp::shutdown();                               
    
    return 0;
}
```

learning_node/src/CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.8)
project(learning_node_cpp)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)

find_package(rclcpp REQUIRED)                                              # 人为添加

add_executable(node_helloworld_class src/node_helloworld_class.cpp)        # 人为添加
ament_target_dependencies(node_helloworld_class rclcpp)                    # 人为添加

install(TARGETS
  node_helloworld_class
  DESTINATION lib/${PROJECT_NAME})                                         # 人为添加

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

learning_node/src/package.xml

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>learning_node_cpp</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="wynhelloworld@gmail.com">wyn</maintainer>
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

### 3. 编译功能包

```shell
cd ~/dev_ws
colcon build   # 编译工作空间的所有功能包
```

### 4. 设置环境变量

```shell
cd ~/dev_ws
source install/local_setup.sh # 仅在当前终端生效
echo " source ~/dev_ws/install/local_setup.sh" >> ~/.bashrc # 所有终端均生效
```

### 5. 运行节点

```shell
ros2 run learning_node node_helloworld
```

![image-20240512170215675](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240512170215675.png)

### 6. 查看节点

- `ros2 node list`：查看正在运行的节点

  ![image-20240512170714292](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240512170714292.png)

- `ros2 node info [options] node_name`：显示节点信息

  ![image-20240512170756327](https://wyn-personal-picture.oss-cn-beijing.aliyuncs.com/img/image-20240512170756327.png)

## 参考链接

[https://book.guyuehome.com/ROS2/2.%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5/2.3_%E8%8A%82%E7%82%B9/](https://book.guyuehome.com/ROS2/2.%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5/2.3_%E8%8A%82%E7%82%B9/)
