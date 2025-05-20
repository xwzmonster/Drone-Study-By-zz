# day01 - how to build own robot

## 关于SDF文件
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
    `<name="1ms>"`：物理配置标识，可自定义
    `<type="ignored>"`：动态引擎类型(物理库)，有Ode、Bullet、Simbody和Dart等。不选择则用`ignored`标记
    `<max_step_size>`：模拟仿真中的每个系统与世界状态交互的最长时间，值越小，仿真精度越高但计算量倍增
    `<real_time_factor>`：模拟时间与实际时间的比值，值 > 1即仿真时间流速快于实际时间，表示加快仿真速度。值 < 1即仿真时间流速慢于实际时间，表示降低仿真速度。

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

