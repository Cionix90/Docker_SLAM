# Docker_SLAM
This repository cointains 2 docker container, docker_simulator and glim. The first one can be used to create a Gazebo SIM world, spawn a teleoperated robot mobile base.
The available mobile base are Omicar (mecanum wheeled kinematic), the Agilex Scout 2.0 (4WD kinematic) and the Agilex Hunter 2.0 (ackerman kinematics).
nvidi   
## Troubleshooting

if is present the compilation error, unrecognise allias GTSAM::GTSAM open the file:
``` /home/punk-opc/Documents/LO_Software/Docker_SLAM/slam_modules/src/glim/CMakeLists.txt ```
and sobstitute "GTSAM::GTSAM" with "gtsam" 

## Launch 
