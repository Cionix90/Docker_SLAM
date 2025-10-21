# Docker_SLAM
This repository provides 2 docker container, docker_simulator and glim. docker_simulator is used to create a Gazebo simulation world and spawn a teleoperated mobile robot base equipped with a 3D LiDAR and an IMU.
The available mobile base are Omicar (mecanum wheeled kinematic), the Agilex Scout 2.0 (4WD kinematic) and the Agilex Hunter 2.0 (Ackerman kinematics). 
**GLIM** is a general, modular **LiDAR-based SLAM framework** capable of:

- **LiDAR–IMU odometry estimation** (GPU and CPU versions)
- **Loop closure detection**
- **Submap & global graph optimization**
- **Multi-session map merging**
- **Offline map visualization**

Its modular architecture separates front-end (odometry), mid-level (submapping), and back-end (global optimization) processes, making it flexible for various robot and sensor setups.  
## GLIM Troubleshooting

“If you encounter a compilation error that does not recognize the alias GTSAM::GTSAM, open the file:
``` Docker_SLAM/slam_modules/src/glim/CMakeLists.txt ```
and sobstitute "GTSAM::GTSAM" with "gtsam" 

## GLIM Configuration Parameters

This document describes the main configuration parameters used in **GLIM (General Localization and Mapping)**.  
Each section corresponds to one of the JSON configuration files used to launch GLIM nodes in ROS 2.

---

###  ROS Configuration — `config_ros.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `acc_scale` | `1.0` | Linear acceleration scaling factor. Set this to **9.80665** if the IMU linear acceleration is expressed in *g* instead of *m/s²* (e.g., Livox LiDARs). |
| `imu_topic` | — | ROS topic name for IMU data input. |
| `points_topic` | — | ROS topic name for LiDAR point cloud input. |
| `image_topic` | — | ROS topic name for camera image input (optional). |

---

### Sensor Configuration — `config_sensors.json`

| Parameter | Description |
|------------|-------------|
| `T_lidar_imu` | Transformation matrix from **IMU** frame to **LiDAR** frame. When the IMU is at rest and its z-axis points upward, the measured linear acceleration should be close to `[0, 0, +9.81]`. <br>Refer to **ROS REP 145** and the GLIM FAQ for frame alignment conventions. |

---

### Preprocessing — `config_preprocess.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `random_downsample_target` | `10000` points | Target number of points after random downsampling. Lowering this value (e.g., to 5000) accelerates computation but reduces map density. |
| `k_correspondences` | `10` points | Number of nearest neighbors used to estimate point covariances. Increase (15–30) for sparse LiDARs like Velodyne VLP-16 to avoid degenerate covariance matrices. |

>  **Tip:**  
> To verify covariance quality, set the GLIM viewer’s `color_mode` to `NORMAL`.  
> If point colors appear uniform on flat planes, covariances are properly estimated.

---

### GPU-Based LiDAR–IMU Odometry — `config_odometry.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `voxel_resolution` | `0.25 m` | Base **VGICP voxel resolution**. Use smaller values (0.1–0.25 m) for indoor environments for higher precision. |
| `voxelmap_levels` | `2` | Number of multi-resolution voxel levels. Increasing improves robustness to large motion displacements. |
| `max_num_keyframes` | `15` | Maximum number of stored keyframes. Increasing reduces drift but increases computation. |
| `keyframe_update_strategy` | `"OVERLAP"` | Strategy for inserting new keyframes:<br>• `OVERLAP`: adaptive metric-based insertion (recommended)<br>• `DISPLACEMENT`: displacement-based (tuned via `keyframe_delta_trans` and `keyframe_delta_rot`)<br>• `ENTROPY`: entropy-based (harder to tune, rarely used). |

---

### CPU-Based LiDAR–IMU Odometry — `config_odometry_cpu.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `registration_type` | `"GICP"` | Registration method, either `"GICP"` or `"VGICP"`. |
| `ivox_resolution` | `0.5 m` | (Used with `"GICP"`) Resolution of iVox voxels. Controls both voxel size and correspondence distance. Increase (~1.0 m) for outdoor operation. |
| `vgicp_resolution` | `0.5 m` | (Used with `"VGICP"`) Resolution of voxelized GICP. Use 0.25–0.5 m indoors and 0.5–2.0 m outdoors. |

---

### LiDAR-Only Odometry — `config_odometry_ct.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `max_correspondence_distance` | `2.0 m` | Maximum correspondence distance for scan-to-scan matching. Smaller values yield stricter but potentially less stable alignments. |

---

### Submapping — `config_sub_mapping.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `enable_optimization` | `true` | Enables local submap optimization. Can be disabled if odometry is already stable to save computation. |
| *(keyframe-related parameters)* | — | Inherit settings from GPU odometry (see `keyframe_update_strategy`, etc.). |

---

### Global Mapping — `config_global_mapping.json`

| Parameter | Default | Description |
|------------|----------|-------------|
| `min_implicit_loop_overlap` | `0.2` | Minimum map overlap ratio required to create a registration error factor for loop closure. |

---

### Common Mapping Parameters

(Shared by **Submapping** and **Global Mapping** modules)

| Parameter | Default | Description |
|------------|----------|-------------|
| `enable_imu` | `true` | Enables IMU-based constraints. Must be set to `false` when running LiDAR-only odometry. |
| `registration_error_factor_type` | `"VGICP_GPU"` | Backend for registration error computation (`"VGICP"` or `"VGICP_GPU"`). |
| `random_sampling_rate` | `1.0` | Sampling rate for points used in global registration. `1.0` disables random sampling (uses all points). |
| `(submap|keyframe)_voxel_resolution` | `0.5 m` | Base voxel resolution for submaps and keyframes. Use smaller values (0.15–0.25 m) for indoor environments. |
| `(submap|keyframe)_voxelmap_levels` | `2` | Number of voxel levels. Set to 2–3 for improved convergence and global consistency. |

---

### Notes

- Always ensure the **LiDAR–IMU extrinsics (`T_lidar_imu`)** are well-calibrated.  
  Misalignment can cause significant drift in both odometry and mapping.
- Choose **GPU odometry** for high-speed motion or large-scale maps.
- Adjust **voxel resolutions** and **keyframe strategies** depending on your environment (indoor vs. outdoor).
- To monitor mapping health, visualize `/odom`, `/map`, and `/trajectory` in **RViz2**.

---

## Containers Start Up 

Following the [istruction](https://github.com/Cionix90/docker_simulator) the containers can be started using the command:
``` docker compose up ```
or 
``` docker compose up -d``` 
to start the container in a deferred way.

To open a bash session in a specific container uses:

``` docker compose exec <container-name> bash ```

From the **docker_simulation** bash session the simulation can be start using:

``` ign gazebo <world-name>```

and the robot can be spawned using:

``` ros2 launch mulinex_ignition/scout_description/hunter_se_description spawn_robot.launch.py ```

From the **GLIM** bash session the slam algorithm can be started using:

```ros2 run glim_ros glim_rosnode --ros-args -p config_path:=$(realpath ./config) ```

After the session mapping the result can be visualize offline using:

```ros2 run glim_ros offline_viewer ```
### References

- **GLIM Official Documentation:** [https://koide3.github.io/glim/](https://koide3.github.io/glim/)  
- **Original Paper:** Koide et al., *Generalized Localization and Mapping: A Modular SLAM Framework for LiDAR-based Robots*  
- **Author:** Koide, K. (AIST, Japan)
