This is the script that is used to start up all the different processes used by the wheelchair. It can be run using the following command wherever the script is located on the computer:
`./startup_script.sh`

2025 Addition to README - Arturo Matlin


Purpose:

The StartupScript repository contains a single shell script (startup_script.sh) that orchestrates the various software components of the autonomous wheelchair system at boot. When executed on the Jetson platform, it prepares the environment, launches microcontroller communication bridges, starts core ROS 2 nodes, visualises LiDAR data and runs the Electron‑based user interface.

What the script does

Environment setup – It sources the default user .bashrc as well as ROS 2 Humble, the main ROS 2 workspace (~/ros2_ws/install/setup.bash), the Livox workspace (~/ws_livox/install/setup.bash) and the micro‑ROS workspace (~/microros_ws/install/setup.bash). It exports CUDA‑related paths and activates Node Version Manager (NVM) to ensure the correct Node.js version is available.

Micro‑ROS agent launching – A helper function watch_and_start_agents monitors the /dev/ttyACM* serial ports and starts a micro_ros_agent in serial mode for each detected microcontroller device, using a baud rate of 115200. The loop stops after two agents are started (one for the motor controller and one for the sensor controller). This bridges the Pico‑based microcontrollers (whose code lives in the MotorMicrocontroller and SensorMicrocontroller repositories) into the ROS 2 ecosystem via micro‑ROS.

Launching main ROS 2 nodes – The script then runs several binaries from the wheelchair_code_module package in parallel:

ros2 run wheelchair_code_module wheelchair – the primary navigation node which subscribes to all sensors and makes driving decisions, as described in the module’s README.

ros2 run wheelchair_code_module obstacle_publisher_node – processes the LiDAR data and publishes obstacle information.

ros2 run wheelchair_code_module temp_monitor – monitors the Jetson’s temperature and controls the fans.
These nodes rely on the custom message definitions in the wheelchair_sensor_msgs repository and expect that workspace to have been built and sourced.

LiDAR driver and visualisation – It launches ros2 launch livox_ros_driver2 rviz_MID360_launch.py to start the Livox driver and open an RViz window showing the Mid‑360 point cloud. livox_ros_driver2 is an external package that publishes /livox/lidar; its output is consumed by both wheelchair_code_module and the LiDAR analysis tools.

Starting the front‑end GUI – The script starts two Node.js processes from the Frontend repository: it runs npm start inside the test-app directory to launch the Electron GUI and runs node middle_man.js inside electron-communication to act as a bridge between ROS 2 and Electron. The front end uses rclnodejs to subscribe to the custom sensor messages defined in wheelchair_sensor_msgs and displays system status, controls, and notifications.

Process management and shutdown – The script records all child process IDs in an array. It sets a trap so that pressing Ctrl‑C triggers a cleanup function which kills all launched processes before exiting. Finally, it waits for all background jobs to finish using wait.

Code dependencies within the organization

The startup script is tightly integrated with other repositories in the TourGuideSeniorDesign organization:

wheelchair_code_module – Provides the core ROS 2 nodes for the wheelchair. These nodes perform LiDAR‑based obstacle detection, fuse sensor data and make navigation decisions. The script runs the wheelchair, obstacle_publisher_node and temp_monitor executables from this package. The module itself depends on the wheelchair_sensor_msgs repository for custom message types.


Frontend – Hosts the Electron‐based user interface and an electron-communication directory containing Node scripts to communicate with ROS 2. The script changes into these directories and runs npm start and node middle_man.js, respectively. The Frontend repository’s README explains how to install Node, build the app, rebuild rclnodejs and generate the custom ROS messages.

livox_ros_driver2 – An external driver (not hosted in this organisation) used to read the Livox Mid‑360 LiDAR. The script launches it via a ROS 2 launch file so that LiDAR data becomes available on /livox/lidar.

wheelchair_sensor_msgs – Defines the custom ROS 2 message types used by both the back end and the front‑end. Although not referenced explicitly in the script, it must be built and sourced for the other packages to compile and run.

MotorMicrocontroller and SensorMicrocontroller – Contain PlatformIO projects for the motor and sensor microcontrollers. These Picos run micro‑ROS clients; the startup_script.sh uses micro_ros_agent to connect to them and expose their topics to the ROS 2 network. Without these agents, the microcontroller data would not reach the Jetson.

Other repositories – The organisation also maintains lidar_analysis and lidar_subscriber (described above) plus several hardware‑oriented repositories (UWB, localization, PCBs, SerialROS, etc.). The startup script does not directly invoke code from these repositories, but developers may use them during development or simulation.

Usage

Clone StartupScript into your home directory on the Jetson and make the script executable (chmod +x startup_script.sh). Before running it, ensure that the wheelchair_code_module, Frontend, wheelchair_sensor_msgs, the micro‑ROS workspaces and livox_ros_driver2 are built and sourced as indicated in the organisation’s READMEs. Run the script with ./startup_script.sh to bring up all components; press Ctrl‑C to cleanly shut them down. The JETSON ALREADY HAS THIS COMPLETED! JUST RUN script.
