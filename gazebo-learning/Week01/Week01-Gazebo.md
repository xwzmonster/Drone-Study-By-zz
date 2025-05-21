# Week01 - Gazebo:how to build own robot

## 关于SDF文件 --- SDF worlds
### 核心基础结构解析与规范

#### Define a world
1、每个SDF world文件都以如下标签开头：
```xml
<?xml version="1.0" ?>
<sdf version="1.8">
    <world name="world_demo">
        ...
        ...
    </world>
</sdf>
```
我们注意到：前两个标签定义了xml和sdf的版本
1、`<?xml version = "xml version"?>`：定义了xml的版本
2、`<sdf version = "sdf verison">`：定义了sdf的版本
3、`<world> ... </world>`中包含所有仿真元素(模型、物理规则光照)


#### Physics
```xml
<!--物理引擎装置-->
<physics name="1ms" type="ignored">
    <max_step_size>0.001</max_step_size>
    <real_time_factor>1.0</real_time_factor>
</physics>
```
这些物理标签：
&ensp;&ensp;&ensp;&ensp;1、`<name="1ms>"`：物理配置标识，可自定义
&ensp;&ensp;&ensp;&ensp;2、`<type="ignored>"`：动态引擎类型(物理库)，有Ode、Bullet、Simbody和Dart等。不选择则用`ignored`标记
&ensp;&ensp;&ensp;&ensp;3、`<max_step_size>`：模拟仿真中的每个系统与世界状态交互的最长时间，值越小，仿真精度越高但计算量倍增
&ensp;&ensp;&ensp;&ensp;4、`<real_time_factor>`：模拟时间与实际时间的比值，值 > 1即仿真时间流速快于实际时间，表示加快仿真速度。值 < 1即仿真时间流速慢于实际时间，表示降低仿真速度。


#### Plugins
##### 插件是指动态加载的代码块，比如
物理插件对于模拟世界动态变化十分重要
```xml
<!--物理插件-->
<plugin
    filename="libignition-gazebo-physics-system.so"
    name="ignition::gazebo::systems::UserCommands">
</plugin>
```

用户命令插件：主要用于创建模型、移动模型，删除模型和其他许多用户命令
```xml
<!--用户命令插件-->
<plugin
    filename="libignition-gazebo-user-commands-system.so"
    name="ignition::gazebo::systems::UserCommands">
</plugin>
```
filename：用于指定动态链接库文件(.so)路径，加载插件功能
name：定义插件的完整命名空间，需与代码中注册的插件名称一致

SceneBroadcaster插件：仿真中的场景状态广播中枢，负责将仿真世界的实时数据(模型位姿、传感器状态、环境参数等)同步到可视化工具和其他子系统。
```xml
<!--SceneBroadcaster插件-->
<plugin
    filename="libignition-gazebo-scene-broadcaster-system.so"
    name="ignition::gazebo::systems::SceneBroadcaster">
</plugin>
```

#### GUI
```xml
<!--GUI标签-->
<gui fullscreen="0">
    ...
    ...
</gui>
```
`<gui>`标签通过集成插件实现界面元素的定制化，例如3D场景渲染、仿真控制按钮、数据监控面板等。这些插件决定了用户在Gazebo中看到的界面和可操作的交互功能。
fullscreen：控制GUI是否全屏模式启动，"0"窗口化显示，"1"全屏

igntiontion-gui有很多插件可供选择。我们将添加那些让我们的世界启动并运行基本功能所必需的插件，比如：
##### Minimal Scene and GzSceneManager plugins
```xml
<!--Minimal Scene plugins-->
<!--3D scene-->
<plugin filename="MinimalScene" name="3D View">
  <gz-gui>
    <title>3D View</title>
    <!--设置窗口标题栏-->

    <property type="bool" key="showTitleBar">false</property>
    <!-- 隐藏窗口标题栏 -->

    <property type="string" key="state">docked</property>
    <!--设置窗口停靠在主界面-->
  </gz-gui>

  <engine>ogre2</engine> <!--选择ogre2渲染引擎-->
  <scene>scene</scene>
  <ambient_light>0.4 0.4 0.4</ambient_light>

  <background_color>0.8 0.8 0.8</background_color>
  <!--背景颜色为浅灰色（RGB 值各 0.8-->

  <camera_pose>-6 0 6 0 0.5 0</camera_pose>
  <!--相机初始位姿：位置 (-6,0,6)，姿态角 (0,0.5弧度≈28.6°,0)-->

  <camera_clip>  <!--控制相机可视范围-->
    <near>0.25</near>
    <!--近裁剪平面：距离相机0.25米内物体不可见-->

    <far>25000</far>
    <!--远裁剪平面：距离相机25000米外物体不可见-->
  </camera_clip> 

<!--GzSceneManager plugins-->
</plugin>
<plugin filename="GzSceneManager" name="Scene Manager">
  <gz-gui>
    <property key="resizable" type="bool">false</property>
    <!--禁止用户调整窗口大小-->

    <property key="width" type="double">5</property>
    <!--设置窗口初始宽度为5单位(像素或比例)-->

    <property key="height" type="double">5</property>
    <!--设置窗口初始高度为5单位(像素或比例)-->

    <property key="state" type="string">floating</property>
    <!--设置窗口浮动显示,floating可拖动到任意位置，docked固定布局，不可拖动-->

    <property key="showTitleBar" type="bool">false</property>
    <!--隐藏窗口标题栏-->
  </gz-gui>
</plugin>
```
MinimalScene：用于渲染3D场景，负责显示机器人、环境模型及物理交互效果。主要有以下子标签：
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;1、engine：指定渲染引擎（如ogre或ogre2），影响图形性能与效果
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;2、scene：绑定场景名称(默认为scene)，与SDF文件中的场景定义关联
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;3、ambient_light：设置全局环境光颜色(RGB值)，影响模型表面亮度
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;4、camera_pose：定义相机的初始位置和朝向(格式：X Y Z Roll Pitch Yaw)
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;5、`<property>`：`<property>`标签用于​​定义 GUI 插件的界面显示属性和行为参数​​，其核心作用是通过键值对（Key-Value）的形式配置界面元素的布局、外观和交互逻辑
&ensp;&ensp;&ensp;&ensp;&ensp;语法结构:`<property key="属性名" type="数据类型">值</property>`。

GzScene3D插件已弃用，来自Fortress版本的`MinimalScene`和`GzSceneManager`负责显示我们世界的3D场景。它具有以下属性（大多数GUI插件都有）：
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;1、show TitleBar如果为真，它将在插件上显示带有`<title>`标签中提到的名称的蓝色标题栏。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;2、state是插件的状态，它可以使用`docked`停靠在它的位置上，也可以是浮动`floating`的。
&ensp;&ensp;&ensp;&ensp;对于渲染引擎，我们可以选择ogre或ogre2。`<ambient_light>`和`<background_color>`指定场景的环境和背景颜色。`<camera_pose>`指定相机的X Y Z位置，然后在滚动俯仰偏航中旋转。


#### World control plugin
```xml
<!-- World control -->
<plugin filename="WorldControl" name="World control">
    <ignition-gui>
        <title>World control</title>
        <property type="bool" key="showTitleBar">false</property>
        <property type="bool" key="resizable">false</property>
        <property type="double" key="height">72</property>
        <property type="double" key="width">121</property>
        <property type="double" key="z">1</property>

        <property type="string" key="state">floating</property>
        <anchors target="3D View">
        <!--将窗口与名为3D View的视图对齐-->
        <line own="left" target="left"/>
        <!--左边缘对齐-->
        <line own="bottom" target="bottom"/>
        <!--底边缘对齐，实现吸附效果-->
        </anchors>
        <!--anchors实现控制面板与3D视图的联动布局-->
    </ignition-gui>

    <play_pause>true</play_pause>
    <step>true</step>
    <start_paused>true</start_paused>
    <service>/world/world_demo/control</service>
    <stats_topic>/world/world_demo/stats</stats_topic>
</plugin>
```
World control：用于控制世界，有一些属性如下：
&ensp;&ensp;&ensp;&ensp;`<play_pause>`：如果为真，左下角将会有播放暂停建
&ensp;&ensp;&ensp;&ensp;`<stats_topic>`：标签指定发布模拟时间和实时等世界统计数据的主题
&ensp;&ensp;&ensp;&ensp;`<step>`：显示单步执行按钮，点击后仿真推进一个步长。
&ensp;&ensp;&ensp;&ensp;`<start_paused>`：如果为真，启动时默认暂停仿真，防止自动运行导致模型失控。
```xml
<service>/world/world_demo/control</service>
<stats_topic>/world/world_demo/stats</stats_topic>
```
定义与ROS系统的通信接口。比如：
&ensp;&ensp;&ensp;&ensp;service：指定 Gazebo 世界控制服务话题，用于接收外部控制指令(如 ROS 的 rosservice call)
&ensp;&ensp;&ensp;&ensp;stats_topic：发布仿真统计信息(如实时因子、迭代次数)的话题名称


#### World stats plugin

```xml
<!-- World statistics -->
<plugin filename="WorldStats" name="World stats">
    <ignition-gui>
        <title>World stats</title>
        <property type="bool" key="showTitleBar">false</property>
        <property type="bool" key="resizable">false</property>
        <property type="double" key="height">110</property>
        <property type="double" key="width">290</property>
        <property type="double" key="z">1</property>

        <property type="string" key="state">floating</property>
        <anchors target="3D View">
        <line own="right" target="right"/>
        <line own="bottom" target="bottom"/>
        </anchors>
    </ignition-gui>

    <sim_time>true</sim_time>
    <real_time>true</real_time>
    <real_time_factor>true</real_time_factor>
    <iterations>true</iterations>
    <topic>/world/world_demo/stats</topic>
</plugin>
```
World stats plugins：负责显示世界统计信息比如`<sim_time`，`real_time`，`real_time`，`real_time_factor`，`iterations`等信息;
使用这些标签，我们可以选择要显示的值，展开右下角以查看这些值。我们可以选择这些值将在哪个`<topics>`上发布。

如何尝试运行这个世界并监听这些话题呢？
1、cd .sdf所在父文件目录下
1、ign gazebo world_demo.sdf

nnd，受不了了，为啥一直没有这个libignition-gazebo-world-stats6-system.so文件啊！！！！





































