# skimap_ros for INDIGO
This package contains the **SkiMap Mapping Framework** described here:


> DE GREGORIO, Daniele; DI STEFANO, Luigi. **SkiMap: An Efficient Mapping Framework for Robot Navigation**. In: *Robotics and Automation (ICRA), 2017 IEEE International Conference on*. [**PDF**](https://dl.dropboxusercontent.com/u/18830822/ICRA17_1818_FI.pdf)


The framework si wrapped in a ROS package to maximize portability but, since it is an *Header-Only* library,
can be easily used elsewhere. This package contains also an implementation of **SlamDunk** algorithm described
in the last section used to track camera 6-DOF pose.

This package is equipped with a sample BAG file: 
[tiago_lar.bag](https://mega.nz/#!QJA2mS6Z!bv67y8nTiQGWT6f5tL05SegwUYxaEAvUuwpLcLc6bSc)
. This is bag was collected by means of a mobile robot with an head-mounting
RGB-D camera. So it provides not only RGB-D frames but also a whole TF tree, including Odometry informations. 
For this reason this bag can be used as source for **SkiMap** mapping using Odometry to track Camera pose or as source 
for **SlamDunk+SkiMap** duet where Camera is tracked by Slam System.

## SkiMap live mapping: *skimap_live.launch*

If the Camera 6-DOF pose is available to us (e.g. the camera is in a eye-to-hand configuration..) we can use **SkiMap**
as a classical Mapping Framework. Furthermore, as described in the paper, if the Global Reference Frames of the Camera Poses 
lies on ground then SkiMap is able to perform also a 2D Map of the environment simultaneously with the 3D Map. 
The attached BAG is an useful example to understand the operation of this node because it is collected with a mobile robot 
with an head-mounted RGB-D camera so the Global Reference frame is certainly on the Ground (Odometry is the Global RF). To
launch the example you have to run two separate commands:


```
1) roslaunch skimap_ros slamdunk_tracker.launch
2) rosbag play tiago_lar.bag --clock
```

If you want to run *skimap_live.launch* in your real context you have to change the Camera Topics parameters and the TF parameters
in the launch file:


```
<!-- Camera topic for RGB and Depth images -->
<param name="camera_rgb_topic" value="/$(arg camera)/rgb/image_raw"/>
<param name="camera_depth_topic" value="/$(arg camera)/depth/image_raw"/>


<!-- Global and Camera TF Names -->
<param name="base_frame_name" value="odom"/>
<param name="camera_frame_name" value="xtion_rgb_optical_frame"/>
```

Remember to set as *base_frame_name* the name of the TF representing the World Frame, or the Fixed Frame, in which the
*camera_frame_name* is represented. Should be noted that **SkiMap** is not responsible to produce these frames but it uses
them to build the map, so the accuracy of the reconstruction depends on the accuracy of them.

## SlamDunk and SkiMap: *slamdunk_tracker.launch*

*Important*: to activate this feature enable build option:

```
option(BUILD_SLAMDUNK "Build Slamdunk Library" OFF)
```


In this node is implemented a Wrapper of the popular **SlamDunk** localization and mapping algorithm, described here:

> FIORAIO, Nicola; DI STEFANO, Luigi. **SlamDunk: affordable real-time RGB-D SLAM**. In: *Workshop at the European Conference on Computer Vision. Springer International Publishing*, 2014. p. 401-414. [**PDF**](http://ai2-s2-pdfs.s3.amazonaws.com/7e9e/191c127144b61d5d5cabac37bbbc27fe7697.pdf)

This is a real-time solution to RGB-D SLAM problem. It enables metric loop closure seamlessly and preserves local consistency by means
of relative bundle adjustment principles. In the first paper *degregorio2017skimap* we described a combination of both systems.
Also in this use case we can exploit the attached bag file: in this case however we will benefit only from the RGB-D Frames
provided in the Bag file because the **SlamDunk** algorithm will provide us a consistent 6-DOF Camera Pose to be used as source 
to build the map of the environment. For proper use of **SkiMap** also here is better to build the Global Reference Frame in
such a way as it lies on the ground; for this reason the camera must get a shot of the ground plane in the very first frame, 
as occurs in the example bag. To launch the complete example you have to run two separate commands:

```
1) roslaunch skimap_ros slamdunk_tracker.launch
2) rosbag play tiago_lar.bag --clock
```

Also here , to adapt this node to your real context, pay attention to the two Camera Topics in the launch file:

```
<!-- Camera topic for RGB and Depth images -->
<param name="camera_rgb_topic" value="/$(arg camera)/rgb/image_raw"/>
<param name="camera_depth_topic" value="/$(arg camera)/depth/image_raw"/>
```

Here you can also disable *SkiMap*:

```
<param name="mapping" value="true"/>
```

Using only the *camera tracking* feature offered by *SlamDunk*. The latter will publish computed camera 6-DOF Pose in a TF tree 
with these two names:

```
<param name="base_frame_name" value="slam_map"/>
<param name="camera_frame_name" value="camera"/>
```
Where *base_frame_name* will be the Global Reference Frame of the Slam system (e.g. usually centered in the first camera pose).

# LICENSE
If you use *SkiMap* in an academic work, please cite:
```
@inproceedings{degregorio2017skimap,
  title={SkiMap: An Efficient Mapping Framework for Robot Navigation},
  author={De Gregorio, Daniele and Di Stefano, Luigi},
  booktitle={Robotics and Automation (ICRA), 2017 IEEE International Conference on},
  year={2017}
}
```

If you use *SlamDunk* in an academic work, please cite:

```
@inproceedings{fioraio2014slamdunk,
  title={SlamDunk: affordable real-time RGB-D SLAM},
  author={Fioraio, Nicola and Di Stefano, Luigi},
  booktitle={Workshop at the European Conference on Computer Vision},
  pages={401--414},
  year={2014},
  organization={Springer}
}
```

