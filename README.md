## Introduction 
This project contains a set of patches and scripts to cross-compile ROS inside a Ubuntu Docker container using a i386 architecture. The compiled ROS files can then be run onboard a Pepper robot, without the need of a tethered computer or superuser privileges. 

## Pre-requirements
Download the NaoQi C++ framework and Softbanks's crosstool chain from [here](http://doc.aldebaran.com/2-5/index_dev_guide.html) and extract them to the users home folder. The crosstool chain ctc-linux64-atom-2.5.2.74 should be a subfolder of the NaoQi C++ framework. Then point the `AL_DIR` and `ALDE_CTC_CROSS` environment variables to their respective paths:
```
export AL_DIR=/home/[user]/naoqi
export ALDE_CTC_CROSS=$AL_DIR/ctc-linux64-atom-2.5.2.74
```

## ROS
### Prepare cross-compiling environment
Clone the project's repository 
``` 
git clone https://github.com/kw90/ros2_pepper.git
cd pepper-ros-install-isolated
```

### Prepare the requirements for ROS 
The following script will create a Docker image and compile Python interpreters suitable for both the host and the robot. 
``` 
./prepare_requirements_ros1.sh 
``` 

### Build ROS dependencies 
Before we actually build ROS for Pepper, there's a bunch of dependencies we'll need to cross compile which are not available in Softbank's CTC: 
- console_bridge 
- uuid 
- poco 
- urdfdom_headers 
- urdfdom 
- tinyxml2 
- SDL 
- SDL_image 
- bullet3 
- yaml-cpp 
- qhull 
- flann 
- PCL 

``` 
./build_ros1_dependencies.sh 
``` 

### Check required ROS packages 
Check and if necessary add required ROS packages from a [ros-gbp release page](https://github.com/ros-gbp) to the file *repos/pepper_ros1.repos* of the form: 
```
[ROS_package]:
     type: git
     url: https://github.com/ros-gbp/[ROS_release-package].git
     version: [specific release version]
``` 

### Build ROS 
Finally, build the whole ROS core and other ROS catkin packages
``` 
./build_ros1.sh 
``` 

### Copy ROS and its dependencies to the robot 
By now you should have the following inside *.ros-root* in the current directory: 
- Python 2.7 built for Pepper (.ros-root/Python-2.7.13) 
- All the dependencies required by ROS (.ros-root/ros1_dependencies) 
- A ROS workspace with ROS Kinetic built for Pepper (.ros-root/ros1_inst) 
- A helper script that will set up the ROS workspace in the robot (.ros-root/setup_ros1_pepper.bash) 

We're going to copy these to the robot, assuming that your robot is connected to your network, type the following: 
``` 
scp -r .ros-root nao@IP_ADDRESS_OF_PEPPER:.ros-root 
``` 

### Run ROS on Pepper 
Now that we have it all in the robot, let's give it a try: 

- SSH into the robot

    ```
    ssh nao@IP_ADDRESS_OF_PEPPER
    ``` 
- Source the setup script

    ```
    source .ros-root/setup_ros1_pepper.bash
    ``` 
- Start *naoqi_driver*, note that *NETWORK\_INTERFACE* may be either **wlan0** or **eth0**, pick the appropriate interface if your robot is connected via wifi or ethernet.

    ```
    roslaunch pepper_bringup pepper_full.launch nao_ip:=IP_ADDRESS_OF_PEPPER roscore_ip:=IP_ADDRESS_OF_PEPPER network_interface:=NETWORK_INTERFACE
    ```

