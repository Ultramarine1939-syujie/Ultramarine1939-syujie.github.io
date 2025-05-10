---
layout: post
title: Xacro模型的使用
subtitle: Each post also has a subtitle
categories: ROS
tags: ROS gazebo
---

### Xacro的使用

realsense_ros_gazebo里提供的.xacro就是个大套娃，为了提取最关键最重要的部分进行urdf/sdf转换，需要将文件进行重构，具体来说就是将xacro中的描述性代码复制到urdf中，随后进行xacro->urdf->sdf的转换

**1 xacro转urdf**
(1) 利用cd命令切换到 xxx.acro 目录下
(2) 执行下面命令即可

```bash
rosrun xacro xacro xxx.xacro > xxx.urdf
```

**2 urdf转sdf**
(1) 利用cd命令切换到 xxx.urdf 目录下
(2) 执行下面命令即可

```shell
 gz sdf -p xxx.urdf > xxx.sdf
```

**修改实例如下：**

修改前：

```xml
<?xml version="1.0"?>
<robot name="test_model" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <xacro:include filename="$(find realsense_ros_gazebo)/xacro/tracker.xacro"/>
  <xacro:include filename="$(find realsense_ros_gazebo)/xacro/depthcam.xacro"/>

  <link name="base_footprint"/>

  <joint name="footprint_joint" type="fixed">
    <parent link="base_footprint"/>
    <child link="base_link"/>
    <origin rpy="0 0 0" xyz="0 0 0"/>
  </joint>

  <link name="base_link">
    <visual>
      <origin rpy="0 0 0 " xyz="0 0 0.005"/>
      <geometry>
          <box size="0.4 0.4 0.01"/>
      </geometry>
    </visual>
    <collision>
      <origin rpy="0 0 0 " xyz="0 0 0.005"/>
      <geometry>
        <box size="0.4 0.4 0.01"/>
      </geometry>
    </collision>
    <inertial>
      <origin rpy="0 0 0 " xyz="0 0 0.005"/>
      <geometry>
        <box size="0.4 0.4 0.01"/>
      </geometry>
      <mass value="100"/>
      <inertia ixx="1.0265" ixy="0" ixz="0" iyy="1.3944999999999999" iyz="0" izz="2.1320000000000001"/>
    </inertial>
  </link>

  <!-- TEST -->

  <!--   <xacro:realsense_d435 sensor_name="d435" parent_link="base_link" rate="10">
    <origin rpy="0 0 0 " xyz="0 0 0.5"/>
  </xacro:realsense_d435>-->

  <xacro:realsense_T265 sensor_name="camera" parent_link="base_link" rate="30.0">
    <origin rpy="0 0 0" xyz="0 0 0.5"/>
  </xacro:realsense_T265> 

  <!-- <xacro:realsense_R200 sensor_name="camera" parent_link="base_link" rate="30.0">
    <origin rpy="0 0 0" xyz="0 0 0.5"/>
  </xacro:realsense_R200> -->


</robot>

```

修改后：

```xml
<?xml version="1.0"?>
<robot name="t265" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <!-- T265 Fisheye:-->
  <xacro:macro name="T265_fisheye" params="sensor_name reference_frame topic rate">
    <gazebo reference="${reference_frame}">
      <sensor name="camera" type="wideanglecamera">
        <camera>
          <horizontal_fov>6.283</horizontal_fov>
          <image>
            <width>800</width>
            <height>848</height>
          </image>
          <clip>
            <near>0.1</near>
            <far>100</far>
          </clip>
          <lens>
            <type>custom</type>
            <custom_function>
              <c1>1.05</c1>
              <c2>4</c2>
              <f>1.0</f>
              <fun>tan</fun>
            </custom_function>
            <scale_to_hfov>true</scale_to_hfov>
            <cutoff_angle>3.1415</cutoff_angle>
            <env_texture_size>512</env_texture_size>
          </lens>
          <always_on>true</always_on>
          <update_rate>30</update_rate>
        </camera>
        <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
          <alwaysOn>true</alwaysOn>
          <updateRate>${rate}</updateRate>
          <cameraName>${sensor_name}/${topic}</cameraName>
          <imageTopicName>image_raw</imageTopicName>
          <cameraInfoTopicName>camera_info</cameraInfoTopicName>
          <frameName>${reference_frame}</frameName>
          <hackBaseline>0.07</hackBaseline>
          <distortionK1>-0.007419134024530649</distortionK1>
          <distortionK2>0.041209351271390915</distortionK2>
          <distortionK3>-0.03811917081475258</distortionK3>
          <distortionT1>0.006366158835589886</distortionT1>
          <distortionT2>0.0</distortionT2>
          <CxPrime>416.00531005859375</CxPrime>
          <Cx>16.00531005859375</Cx>
          <Cy>403.38909912109375</Cy>
        </plugin>
      </sensor>
    </gazebo>
  </xacro:macro>

  <!-- INTEL REALSENSE T265 -->
  <xacro:macro name="realsense_T265" params="sensor_name parent_link rate *origin">

    <joint name="${sensor_name}_odom_frame_joint" type="fixed">
        <parent link="${parent_link}"/>
        <child link="${sensor_name}_odom_frame"/>
        <xacro:insert_block name="origin"/>
    </joint>

    <joint name="${sensor_name}_pose_frame_joint" type="fixed">
        <parent link="${sensor_name}_odom_frame"/>
        <child link="${sensor_name}_pose_frame"/>
        <origin rpy="0 0 0" xyz="0 0 0"/> <!-- check on real hw -->
    </joint>
    <link name="${sensor_name}_odom_frame"/>

    <link name="${sensor_name}_pose_frame">
        <visual>
            <origin rpy="1.57 0 1.57" xyz="0 0 0"/>
            <geometry>
                <mesh filename="package://realsense_ros_gazebo/meshes/realsense_t265.stl" scale="0.001 0.001 0.001"/>
            </geometry>
        </visual>
        <collision>
            <geometry>
                <box size="0.013 0.108 0.024"/>
            </geometry>
        </collision>
        <inertial>
            <mass value="0.055"/>
            <origin rpy="0 0 0" xyz="0 0 0"/>
            <inertia ixx="9.108e-05"
                     ixy="0"
                     ixz="0"
                     iyy="2.51e-06"
                     iyz="0"
                     izz="8.931e-05"/>
        </inertial>
    </link>

    <joint name="${sensor_name}_joint" type="fixed">
        <parent link="${sensor_name}_pose_frame"/>
        <child link="${sensor_name}_link"/>
        <origin rpy="0 0 0" xyz="0 0 0"/> <!-- check on real hw -->
    </joint>
    <link name="${sensor_name}_link"/>

    <joint name="${sensor_name}_fisheye1_joint" type="fixed">
        <parent link="${sensor_name}_link"/>
        <child link="${sensor_name}_fisheye1_frame"/>
        <origin rpy="0 0 0" xyz="0 0.042 0"/>
    </joint>
    <link name="${sensor_name}_fisheye1_frame"/>

    <joint name="${sensor_name}_fisheye1_optical_joint" type="fixed">
        <parent link="${sensor_name}_fisheye1_frame"/>
        <child link="${sensor_name}_fisheye1_optical_frame"/>
        <origin rpy="0 0 0" xyz="0.01 0 0"/>
    </joint>
    <link name="${sensor_name}_fisheye1_optical_frame"/>
    <xacro:T265_fisheye
      sensor_name="${sensor_name}"
      reference_frame="${sensor_name}_fisheye1_optical_frame"
      topic="fisheye1"
      rate="${rate}">
    </xacro:T265_fisheye>

    <joint name="${sensor_name}_fisheye2_joint" type="fixed">
        <parent link="${sensor_name}_link"/>
        <child link="${sensor_name}_fisheye2_frame"/>
        <origin rpy="0 0 0" xyz="0 -0.022 0"/>
    </joint>
    <link name="${sensor_name}_fisheye2_frame"/>

    <joint name="${sensor_name}_fisheye2_optical_joint" type="fixed">
        <parent link="${sensor_name}_fisheye2_frame"/>
        <child link="${sensor_name}_fisheye2_optical_frame"/>
        <origin rpy="0 0 0" xyz="0.01 0 0"/>
    </joint>
    <link name="${sensor_name}_fisheye2_optical_frame"/>
    <xacro:T265_fisheye
      sensor_name="${sensor_name}"
      reference_frame="${sensor_name}_fisheye2_optical_frame"
      topic="fisheye2"
      rate="${rate}">
    </xacro:T265_fisheye>

    <joint name="${sensor_name}_gyro_joint" type="fixed">
        <parent link="${sensor_name}_link"/>
        <child link="${sensor_name}_gyro_frame"/>
        <origin rpy="0 0 0" xyz="0 0 0"/> <!-- check on real hw -->
    </joint>
    <link name="${sensor_name}_gyro_frame"/>

    <gazebo reference="${sensor_name}_gyro_frame">
        <gravity>true</gravity>
        <sensor name="${sensor_name}_imu" type="imu">
            <always_on>true</always_on>
            <update_rate>${rate}</update_rate>
            <visualize>false</visualize>
            <plugin filename="libgazebo_ros_imu_sensor.so" name="imu_plugin">
                <topicName>${sensor_name}/gyro/sample</topicName>
                <bodyName>${sensor_name}_pose_frame</bodyName>
                <updateRateHZ>${rate}</updateRateHZ>
                <gaussianNoise>0.000001</gaussianNoise>
                <xyzOffset>0 0 0</xyzOffset>
                <rpyOffset>0 0 0</rpyOffset>
                <frameName>${sensor_name}_link</frameName>
            </plugin>
        </sensor>
    </gazebo>

    <joint name="${sensor_name}_accel_joint" type="fixed">
        <parent link="${sensor_name}_link"/>
        <child link="${sensor_name}_accel_frame"/>
        <origin rpy="0 0 0" xyz="0 0 0"/> <!-- check on real hw -->
    </joint>
    <link name="${sensor_name}_accel_frame"/> <!-- dummy -->

    <gazebo> <!-- odometry -->
        <plugin name="p3d_base_controller" filename="libgazebo_ros_p3d.so">
            <alwaysOn>true</alwaysOn>
            <updateRate>${rate}</updateRate>
            <bodyName>${sensor_name}_odom_frame</bodyName>
            <topicName>${sensor_name}/odom/sample</topicName>
            <gaussianNoise>0.001</gaussianNoise>
            <frameName>world</frameName>
            <xyzOffsets>0 0 0</xyzOffsets>
            <rpyOffsets>0 0 0</rpyOffsets>
        </plugin>
    </gazebo>

  </xacro:macro> 

  <link name="base_footprint"/>

  <joint name="footprint_joint" type="fixed">
    <parent link="base_footprint"/>
    <child link="base_link"/>
    <origin rpy="0 0 0" xyz="0 0 0"/>
  </joint>

  <link name="base_link">
  </link>

  <xacro:realsense_T265 sensor_name="camera" parent_link="base_link" rate="30.0">
    <origin rpy="0 0 0" xyz="0 0 0.5"/>
  </xacro:realsense_T265> 


</robot>

```

