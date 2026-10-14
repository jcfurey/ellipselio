<div align="center">
    <h1>EllipseLIO</h1>
    <a href="https://github.com/v4rl-ucy/ellipselio"><img src="https://img.shields.io/badge/-C++-blue?logo=cplusplus" /></a>
    <a href="https://github.com/v4rl-ucy/ellipselio"><img src="https://img.shields.io/badge/ROS2-blue" /></a>
    <a href="https://github.com/v4rl-ucy/ellipselio"><img src="https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black" /></a>
    <a href="https://github.com/v4rl-ucy/ellipselio/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-green.svg" alt="MIT License" /></a>
    <br />
    <br />
    <a href="https://youtu.be/eIZ8CK4TAuA">Video</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/v4rl-ucy/ellipselio/blob/main/README.md">Install</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="http://arxiv.org/abs/2605.21150">arXiv</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://ieeexplore.ieee.org/document/11661668">IEEE</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/v4rl-ucy/ellipselio/issues">Report Issues</a>
  <br />
  <br />
  <p align="center"><img src=ellipselio.gif alt="animated" /></p>

  [EllipseLIO][arXivlink] is an **Adaptive LiDAR Inertial Odometry Approach with an Ellipsoid Representation**

  🎉 EllipseLIO was awarded **3rd Prize 🥉 in the COMFORT Localization Challenge** at the IROS 2026 workshop [Data in Field Robotics][comfortLink] 
</div>

[arXivlink]: http://arxiv.org/abs/2605.21150
[IEEElink]: https://ieeexplore.ieee.org/document/11661668
[comfortLink]: https://jonasfrey96.github.io/data_in_field_robotics/

## ROS 2 Humble, Jazzy, and Lyrical

### Build

```sh
mkdir -p ~/colcon_ws/src
cd ~/colcon_ws/src
git clone --branch cam_wip https://github.com/jcfurey/ellipselio.git
cd ..
colcon build --packages-select ellipselio --cmake-args -DCMAKE_BUILD_TYPE=Release --symlink-install
source ~/colcon_ws/install/setup.bash
```

### ROS 2 Lyrical in an existing workspace

On Ubuntu 26.04 with ROS 2 Lyrical installed, run from the workspace root:

```sh
source /opt/ros/lyrical/setup.bash
rosdep install --from-paths src/ellipselio --ignore-src --rosdistro lyrical -y
colcon build --base-paths src/ellipselio --packages-select ellipselio \
  --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

The build uses `ament_cmake_auto` to link the declared ROS dependencies and
selects the available cv_bridge and TF2 headers directly. Only PCL common is
loaded, avoiding the optional VTK visualization dependencies. Lyrical uses
`component_container --executor-type multi-threaded`; older releases retain
`component_container_mt`. Published frame IDs omit leading slashes for TF2.

### TestTrack Ouster recording

[`config/os128_testtrack.yaml`](config/os128_testtrack.yaml) configures the
June 26, 2026 shredder-raphael TestTrack recording: 128 x 1024 Ouster scans at
10 Hz with approximately 90 degrees of vertical field of view, and the
built-in IMU on `/sensor/lidar/lidar0/{points,imu}`. The LiDAR to
IMU transform comes from the recording's `/tf_static`; IMU noise values are
initial defaults rather than a calibration fitted to this recording. Cameras
and ellipsoid markers are disabled; map, scan, odometry, analytics, and TF
outputs are enabled.

After sourcing the workspace, start EllipseLIO in one terminal:

```sh
export OMP_NUM_THREADS=8
ros2 launch ellipselio ellipselio_standalone.launch.py \
  config_file:=os128_testtrack.yaml rviz:=false
```

In a second terminal, source ROS and the workspace and play a short segment:

```sh
source /opt/ros/lyrical/setup.bash
source install/setup.bash
ros2 bag play \
  RESPLE_dataset/2026-06-26_shredder-raphael_TestTrack_20-08-24_UTC/optimized_lidar_imu_tf \
  --clock 400 --rate 0.5 --playback-duration 30 \
  --topics /sensor/lidar/lidar0/points /sensor/lidar/lidar0/imu
```

Use `rviz:=true` for visualization, and omit `--playback-duration 30` to play
the full recording. The playback command selects the sensor topics so the
recording's original robot TF tree is excluded. `OMP_NUM_THREADS` controls
OpenMP parallelism; eight threads are a starting point for this workstation.

[`config/os64_0705.yaml`](config/os64_0705.yaml) supplies the separate OS1-64
calibration for the local `07052026_4_an` tunnel recording: 64 x 1024 at 10 Hz,
42.22-degree vertical field of view, and a 125 Hz IMU. Its raw packets need an
Ouster decoder publishing `/ouster/points` and `/ouster/imu`; the surrounding
workspace provides `scripts/ellipselio_0705.launch.py` for this replay. The
decoder retains sensor acquisition timing and applies one offset to align
LiDAR and IMU with the bag's epoch. IMU noise values remain initial defaults.

### Run standalone with a bag file

```sh
ros2 launch ellipselio ellipselio_standalone.launch.py config_file:=<config_file_name>
ros2 bag play --clock <imu_rate> <bag_folder> --topics <lidar_topic> <imu_topic>
```

When running a dataset with raw Livox LiDAR data (e.g., GEODE Gamma) you also need to run a separate [node][converterLink] to convert the custom Livox messages to standard PointCloud2 messages.

[converterLink]: https://github.com/v4rl-ucy/livox_to_pointcloud2

### Included dataset configs

| Config file | Dataset |
| --- | --- |
| [`config/os128_ncd.yaml`](config/os128_ncd.yaml) | [`Newer College Multi-Cam`](https://ori-drs.github.io/newer-college-dataset/multi-cam/) |
| [`config/os64_ncd.yaml`](config/os64_ncd.yaml) | [`Newer College Stereo-Cam`](https://ori-drs.github.io/newer-college-dataset/stereo-cam/) |
| [`config/qt64_spires.yaml`](config/qt64_spires.yaml) | [`Oxford Spires`](https://dynamic.robots.ox.ac.uk/datasets/oxford-spires/) |
| [`config/vlp16_bot.yaml`](config/vlp16_bot.yaml) | [`BotanicGarden`](https://github.com/robot-pesg/BotanicGarden) |
| [`config/vlp16_geode.yaml`](config/vlp16_geode.yaml) | [`GEODE Alpha`](https://thisparticle.github.io/geode/) |
| [`config/os64_geode.yaml`](config/os64_geode.yaml) | [`GEODE Beta`](https://thisparticle.github.io/geode/) |
| [`config/avia_geode.yaml`](config/avia_geode.yaml) | [`GEODE Gamma`](https://thisparticle.github.io/geode/) |
| [`config/vlp16_graco.yaml`](config/vlp16_graco.yaml) | [`GRACO`](https://github.com/SYSU-RoboticsLab/GrAco) |
| [`config/vlp16_grandtour.yaml`](config/vlp16_grandtour.yaml) | [`GrandTour VLP-16`](https://grand-tour.leggedrobotics.com/dataset) |
| [`config/xt32_grandtour.yaml`](config/xt32_grandtour.yaml) | [`GrandTour XT-32`](https://grand-tour.leggedrobotics.com/dataset) |
| [`config/mid360_grandtour.yaml`](config/mid360_grandtour.yaml) | [`GrandTour Mid-360`](https://grand-tour.leggedrobotics.com/dataset) |

### Run standalone with live data

```sh
ros2 launch ellipselio ellipselio_standalone.launch.py config_file:=<config_file_name> use_sim_time:=false
```

### Publication control

Published outputs can be enabled or disabled independently in the YAML
configuration. Disabled outputs skip their publication timers and
message-conversion work. Internal odometry and mapping still run normally.

```yaml
publish:
    map: true
    scan: true
    markers: true
    odometry: true
    analytics: true
    tf: true
```

| Parameter | Output |
| --- | --- |
| `publish.map` | `/cloud_map` |
| `publish.scan` | `/cloud_scan` |
| `publish.markers` | `/visualization_marker` |
| `publish.odometry` | `/ellipselio_odom` |
| `publish.analytics` | `/analytics` |
| `publish.tf` | `/tf` |

For example, a headless run that only publishes odometry and TF can use:

```yaml
publish:
    map: false
    scan: false
    markers: false
    odometry: true
    analytics: false
    tf: true
```

The publication settings are read at node startup, so restart the node after
changing them.

## :pencil: Citation

If you use EllipseLIO in your work please cite our RA-L [paper][IEEELink]
```
@article{border2026ellipselio,
   author = {Rowan Border and Margarita Chli},
   doi = {10.1109/LRA.2026.3726372},
   issue = {10},
   journal = {IEEE Robotics and Automation Letters},
   pages = {11474-11481},
   title = {EllipseLIO: Adaptive LiDAR Inertial Odometry with an Ellipsoid Representation},
   volume = {11},
   year = {2026}
}

```

## :pray: Acknowledgements

Many thanks to the authors of [FAST-LIO2][fastliolink], [IKFoM][ikfomlink], and [i-Octree][ioctreelink] for open-sourcing their work, which made the development of EllipseLIO possible. 

[fastliolink]: https://github.com/hku-mars/FAST_LIO
[ikfomlink]: https://github.com/hku-mars/IKFoM
[ioctreelink]: https://github.com/zhujun3753/i-octree

## :mailbox: Contact information

If you have any questions, please do not hesitate to contact
* [Rowan Border][rblink] :envelope: rborder `dot` robots `at` gmail `dot` com

[rblink]: https://github.com/rowanborder
