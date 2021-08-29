### Ubuntu基本小操作：

| 小代码                              | 备注                                         |
| :---------------------------------- | :------------------------------------------- |
| pwd                                 | 查看该终端打开的路径                         |
| cd /home                            | 进入home目录                                 |
| roscd 功能包名                      | 直接进入该功能包目录                         |
| cd ..                               | 退回上一级目录                               |
| mkdir test_folder                   | 在该目录级下创建一个文件夹（make direction） |
| touch test_file                     | 建一个文档（touch）                          |
| ls                                  | 查看当前目录下的内容，空目录则无反应         |
| mv test_file /home/ddyr             | 移动(move)  文件  到路径                     |
| cp test_file test_folder/test_file2 | copy 文件 路径/(新名字)                      |
| rm test_file                        | 删除remove 文件(文档)                        |
| rm -r test_folder                   | 删除一个文件夹(目录)   -r遍历文件夹中内容    |

### ROS

#### 基本操作解释

rospack，roscd，rosls，roscp，rosmsg，rossrv，rosmake

| sudo                | 提升权限                                                     |
| :------------------ | :----------------------------------------------------------- |
| sudo atp-get undate | apptation升级    中途想暂停可以按ctrl+c                      |
| rm --h           | 帮助列表，或直接回车                                         |
| rosbag              | 话题记录工具，可记录所有话题数据并保存下来，下次用时复现出来 |

`rosbag record -a -O 压缩包名`
记录所有数据（all -a）成一个压缩包（-O），回车后进行操作再ctrl+c暂停，如：rosbag record -a -O cmd_record  ，默认保存再home下。

复现，关闭/暂停之前终端，启动roscore和仿真器节点，不启动键盘控制节点
`rosbag play 压缩包名.bag`复现前一步记录的内容，小乌龟开始动（rosbag play cmd_record.bag）
`rosrun [package_name] [node_name]`  rosrun 功能包名 节点名

`rosed [package_name] [filename]`
直接通过软件包名编辑包中的文件，而无需键入完整路径，如：rosed roscpp Logger.msg，编辑roscpp软件包中的Logger.msg文件

`rqt_graph`
调出查看节点及其间通讯关系全貌，如：
/turtlesim-仿真器节点（椭圆）； /teleop_turtle-键盘控制节点；中间箭头为话题通讯（话题）

#### 创建工作空间
操作|备注
:--|:--
在/home下打开终端|
mkdir catkin_ws|该名字可自定义
cd catkin_ws/|
mkdir src|代码空间，必须叫src
cd src/|
（上也可打成：|
mkdir -p /catkin_ws/src|-p连续创建
cd /catkin_ws/src）|
catkin_init_workplace|初始化该文件夹为工作空间属性，src中新增一个功能包原码
cd ..|回到catkin_ws根目录下
catkin_make|编译工作空间
此时新增build（编译空间）和devel（开发空间）两个文件夹|
catkin_make install|产生install（安装空间）文件夹
src（代码空间）|放功能包源码（功能包为放置源码的最小单元，代码必须放入一个个功能包中）
install（安装空间）|放最终生成的可执行文件，及开发集成后，发布给客户使用的文件
devel（开发空间）|放开发过程中的可执行文件及一些库，与install类似
build（编译空间）|中间文件，二进制，基本用不上

#### 创建功能包
操作|备注
:--|:--
cd src/|在src中创建功能包
catkin_create_pkg 功能包名 依赖1 依赖2 依赖3...|如：catkin_create_pkg test_pkg roscpp rospy std_msgs(标准消息)
进入功能包中有四个块块，|
其中src文件夹用于放代码文件，include放头文件，|
另两个为基本文件，标志此文件夹为功能包|
cd ..|回到catkin_ws根目录下
catkin_make|编译功能包，编译出显示已有一个叫test_pkg的功能包
rospack depends1 test_pkg|可查看一级依赖，这些依赖关系存储在package.xml文件中。
rospack depends test_pkg|递归查看所有依赖关系，包括如rospy的间接依赖

#### 环境变量（工作空间下）
`source devel/setup.bash`	   运行程序前先设置环境变量，才能让系统找到此工作空间及其功能包
`echo $ROS_PACKAGE_PATH`		查看功能包路径，如：`/home/ddyr/catkin_ws/src：/opt/ros/kinetic/share`

或复制`source devel/setup.bash`，进到主文件夹，按ctrl+h显示隐藏文件，打开 .bashrc，粘贴在最后，改成：`source /home/ddyr/catkin_ws/devel/setup.bash`
以后可不用每次在终端里输入source创建环境变量，重启终端生效

#### 节点

节点  是一个能执行特定工作任 务的工作单元，并且能够相互通信，从而实现一个机器人系统整体的功能
rosnode|节点相关，可直接按回车出提示
:--|:-

`rosrun turtlesim turtlesim_node __name:=my_turtle`
\__name:=		重映射参数，改变节点名，turtlesim—>my\_turtle


#### 话题

rostopic|获取ROS话题的信息，可按回车出提示
:--|:-

`rosmsg show geometry_msgs/Twist`				查看Twist消息的数据结构（ros已定义好的）

若在 pub后 或 消息类型后 加一个" -r 10 "，以每秒10赫兹的频率发布pub，则可出现循环运动；pub后加一个" -1 "，rostopic会只发布一条消息，然后退出。
如：`rostopic pub -1 /turtle1/cmd_vel geometry_msgs/Twist -- '[2.0, 0.0, 0.0]' '[0.0, 0.0, 1.8]'`		
--（两个破折号）用来告诉选项解析器，表明之后的参数都不是选项

##### 例：
mkdir msg			在功能包中创建一个文件夹msg信息（必须叫msg，CMakeList中写了）
cd msg
touch Person.msg	   创建一个Person.msg文档
写入一段代码，为该消息的结构：
```
string name				
uint8 sex
uint8 age
uint8 unknown=0
uint8 male=1
uint8 famale=2
```
打开该功能包的 package.xml，加入如下两句（添加ros功能包依赖）：
  <build_depend>message_generation</build_depend>		#编译依赖
  <exec_depend>message_runtime</exec_depend>		#执行依赖

打开CMakeList.txt，添加编译选项
   1.find_package(...+message_generation)
   2.在Declare ROS dynamic reconfigure parameters框框之前添加如下两行：
        `add_message_files(FILES Person.msg)
    generate_messages(DEPENDENCIES std_msgs)`

   3. 黑体"catkin_package(“中，第三排：#去掉，末尾加message_runtime

回到catkin_ws工作空间根目录下catkin_make进行编译
编译生成的头文件在 /catkin_ws/devel/include/learning_topic/Person.h

在功能包的src文件夹中新建两个cpp文档：

###### 发布者cpp		person_publisher.cpp
```cpp
#include <ros/ros.h>
#include "learning_topic/Person.h"		//消息类型头文件

int main(int argc, char **argv)
{
    // ROS节点初始化
    ros::init(argc, argv, "person_publisher");//参数，参数，节点名(不可重复)
    
    // 创建节点句柄
    ros::NodeHandle n;		//管理节点资源
    
    // 向ROS Master注册节点信息，包括发布的话题名和话题中的消息类型
    // 创建一个Publisher名为person_info_pub，发布名为/person_info的topic，消息类型为learning_topic::Person，队列长度10。调用句柄中的advertise()方法（返回值类型为ros::Publisher）。
    ros::Publisher person_info_pub = n.advertise<learning_topic::Person>("/person_info", 10);
    
    // 设置循环的频率
    ros::Rate loop_rate(1);
    
    int count = 0;
    while (ros::ok())
    	/*ros::ok()在以下情况会返回false：
		收到SIGINT信号（Ctrl+C）
		被另一个同名的节点踢出了网络
		ros::shutdown()被程序的另一部分调用
		所有的ros::NodeHandles都已被销毁
		一旦ros::ok()返回false, 所有的ROS调用都会失败*/
    {
        // 初始化learning_topic::Person类型的消息
    	learning_topic::Person person_msg;
		person_msg.name = "Tom";
		person_msg.age  = 18
		person_msg.sex  = learning_topic::Person::male;//msg中已规定male=1
        // 发布消息
		person_info_pub.publish(person_msg);//调用Publisher::publish()方法
       	ROS_INFO("Publish Person Info: name:%s  age:%d  sex:%d", person_msg.name.c_str(), person_msg.age, person_msg.sex);

        // 按照循环频率延时
        loop_rate.sleep();
    }
    return 0;
}
```

###### 订阅者cpp		person_subscriber.cpp
```cpp
#include <ros/ros.h>
#include "learning_topic/Person.h"

// 接收到订阅的消息后，会进入消息回调函数
void personInfoCallback(const learning_topic::Person::ConstPtr& msg)
{
    // 将接收到的消息打印出来
    ROS_INFO("Subcribe Person Info: name:%s  age:%d  sex:%d", msg->name.c_str(), msg->age, msg->sex);
}

int main(int argc, char **argv)
{
    // 初始化ROS节点
    ros::init(argc, argv, "person_subscriber");

    // 创建节点句柄
    ros::NodeHandle n;

    // 创建一个Subscriber，订阅名为/person_info的topic，注册回调函数personInfoCallback
    ros::Subscriber person_info_sub = n.subscribe("/person_info", 10, personInfoCallback);

    // 循环等待回调函数
    ros::spin();
    
    return 0;
}
```

在CMakeLists.txt中加入如下六句：

```
add_executable(person_publisher src/person_publisher.cpp)
target_link_libraries(person_publisher ${catkin_LIBRARIES})
add_dependencies(person_publisher ${PROJECT_NAME}_generate_messages_cpp)

add_executable(person_subscriber src/person_subscriber.cpp)
target_link_libraries(person_subscriber ${catkin_LIBRARIES})
add_dependencies(person_subscriber ${PROJECT_NAME}_generate_messages_cpp)
```
回到 catkin_ws 进行 catkin_make 编译
roscore								  								 终端一
rosrun learning_topic person_subscriber		终端二，进入等待
rosrun learning_topic person_publisher		  终端三，回车后开始发布数据，subscriber接收到数据，二者持续通讯，不断在终端打印出该person的消息
此时关闭roscore（rosmaster帮助节点创建连接，创建成功后作用不再存在），两节点不受影响

#### 服务
`rosservice type /服务名`				查看服务类型

如：rosservice type /clear，回std_srvs/Empty，服务的类型为empty（空）,表明调用这个服务时不需要参数；

又如：rosservice type /spawn，回

```
float32 x
float32 y
float32 theta
string name
---                  
string name
```
`rosservice call /服务名 参数`
对某一服务的调用，发布请求，后按两下tab可出现更改内容数据，如：

`rosservice call /spawn 2 2 0.2 ""`  

即可新增一只小海龟，未命名，回 name: turtle2 （自动生成的名字）

##### 例：
在功能包里新建一个文件夹 srv
在srv文件夹中写入一个文件 Person.srv ,内容如下：
```
string name				
uint8 sex
uint8 age
uint8 unknown=0
uint8 male=1
uint8 famale=2			
---				三条短线之上为request数据，以下为response数据。为服务与话题的区别。
string result
```
打开learning_service/package.xml，加入如下两句（添加ros功能包依赖）：
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>

打开CMakeList.txt，添加编译选项
    1.find_package(...+message_generation)
    2.在Declare ROS dynamic reconfigure parameters框框之前添加如下两行：
        `add_service_files(FILES  Person.srv)
    generate_messages(DEPENDENCIES std_msgs)`
    3. build框框下，黑体"catkin_package(“中，第三排：#去掉，末尾加message_runtime	

回到catkin_ws工作空间根目录下catkin_make进行编译
编译生成的头文件在 /catkin_ws/devel/include/learning_service		
包含有三个头文件，Person.h为总和头，另两个为---分开的两部分request和response

###### 客户端cpp		person_client.cpp
```cpp
#include <ros/ros.h>
#include "learning_service/Person.h"

int main(int argc, char** argv)
{
    // 初始化ROS节点
	ros::init(argc, argv, "person_client");

    // 创建节点句柄
	ros::NodeHandle node;

    // 发现/show_person服务后，创建一个服务客户端，连接名为/show_person的service
	ros::service::waitForService("/show_person");
	ros::ServiceClient person_client = node.serviceClient<learning_service::Person>("/show_person");

    // 初始化learning_service::Person的请求数据
	learning_service::Person srv;
	srv.request.name = "Tom";
	srv.request.age  = 20;
	srv.request.sex  = learning_service::Person::Request::male;

    // 请求服务调用
	ROS_INFO("Call service to show person[name:%s, age:%d, sex:%d]", 
			 srv.request.name.c_str(), srv.request.age, srv.request.sex);
	person_client.call(srv);

	// 显示服务调用结果
	ROS_INFO("Show person result : %s", srv.response.result.c_str());

	return 0;
};
```

###### 服务端cpp		person_server.cpp
```cpp
#include <ros/ros.h>
#include "learning_service/Person.h"

// service回调函数，输入参数req，输出参数res
bool personCallback(learning_service::Person::Request  &req, learning_service::Person::Response &res)
{
    // 显示请求数据
    ROS_INFO("Person: name:%s  age:%d  sex:%d", req.name.c_str(), req.age, req.sex);

	// 设置反馈数据
	res.result = "OK";
    return true;
}

int main(int argc, char **argv)
{
    // ROS节点初始化
    ros::init(argc, argv, "person_server");

    // 创建节点句柄
    ros::NodeHandle n;

    // 创建一个名为/show_person的server，注册回调函数personCallback
    ros::ServiceServer person_service = n.advertiseService("/show_person", personCallback);

    // 循环等待回调函数
    ROS_INFO("Ready to show person informtion.");
    ros::spin();

    return 0;
}
```

在CMakeLists.txt中加入如下六句：
```
add_executable(person_publisher src/person_publisher.cpp)
target_link_libraries(person_publisher ${catkin_LIBRARIES})
add_dependencies(person_publisher ${PROJECT_NAME}_generate_messages_cpp)

add_executable(person_subscriber src/person_subscriber.cpp)
target_link_libraries(person_subscriber ${catkin_LIBRARIES})
add_dependencies(person_subscriber ${PROJECT_NAME}_generate_messages_cpp)
```
回到 catkin_ws 进行 catkin_make 编译
roscore					终端一
rosrun learning_service person_server	终端二，进入等待
rosrun learning_service person_client	终端三，client发送请求，server做出显示反馈OK，客户端收到OK
请求一次，反馈一次

#### launch

通过 launch文件以及 roslaunch 命令可以一次性启动多个 node，并且可以设置丰富的参数。

用 roslaunch 命令启动 launch 文件至少有两种方式：

1.借助 ros package 路径启动

```bash
roslaunch  pkg_name  launchfile_name.launch 
```

2.直接给出 launch 文件的绝对路径

```bash
roslaunch path_to_launchfile
```

一个简单的launch文件：

```xml
<launch>
    <node pkg="package_name" type="executable_file" name="node_name1"/>
    <node pkg="another_package" type="another_executable" name="another_node"></node>
    ...
</launch>
```

其中

- `pkg` 是节点所在的 package 名称
- `type`是 package 中的可执行文件，如果是 python 或者 Julia 编写的，就可能是 .py 或者 .jl 文件，如果是 c++ 编写的，就是源文件编译之后的可执行文件的名字。
- `name`是节点启动之后的名字, 每一个节点都要有自己独一无二的名字。

#### Qt工具箱
操作|备注
:--|:--
`roscore`|终端1
`rosrun turtlesim turtlesim_node`|终端2
`rosrun turtlesim turtle_teleop_key`|终端3
rqt_|+2tab
rqt_console|显示所有报错信息	`rosrun rqt_console rqt_console`
rqt_logger_level|在节点运行时改变输出信息的详细级别
rqt_plot|线性显示参数变化	`rosrun rqt_plot rqt_plot`

#### rviz工具（数据显示平台，需要有数据）

`rosrun rviz rviz`	打开rviz

#### Gazebo（仿真平台，自己创建数据）
