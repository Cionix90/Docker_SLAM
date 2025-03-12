# Docker_SLAM

## Troubleshooting

if is present the compilation error, unrecognise allias GTSAM::GTSAM open the file:
``` /home/punk-opc/Documents/LO_Software/Docker_SLAM/slam_modules/src/glim/CMakeLists.txt ```
and sobstitute "GTSAM::GTSAM" with "gtsam" 

## Launch 
To start the simulation in turtle_arena.sdf launch:
``` ros2 launch mulinex_ignition gz_harmonic_sim_w_rbt.launch.py world_name:=turtle_arena.sdf```
To launch the slam node use:

```  ros2 run glim_ros glim_rosnode --ros-args -p  config_path:=/home/ros/docker_slam_ws/glim_config ```