# RustBot
Software tools for the project Sistemas Embarcados de Vistoria (SEV). We refer to the hardware platform as RustBot because the AVT Mako cameras look kind of rusty (sorry for the lack of creativity).

## Table of Contents

* [The Robot](#therobot)
* [Installation](#installation)
* [Connecting via ssh](#connectingssh)
* [Usage](#usage)
* [Calibration](#calibration)
* [Tunning Stereo Parameters](#tunningparameters)
* [Saving a Bag File](#savingabagfile)
* [Playing Back Data](#playingbackdata)
* [Stereo from a Bagfile](#stereobagfile)
* [Publishing Data Using ZMQ](#publishingdatazmq)
* [Finding IP Address of Cameras](#findingcameraip)
* [ZeroMQ + Google Protocol Buffers Tutorial](#zeromqtutorial)
* [Compile Google Protocol Buffers Messages](#compilemessages)

## <a name="therobot"></a>The Robot

The RustyBot is a stereo vision system with two AVT Mako cameras mounted on a plate. The baseline is around 0.1 meters, and the cameras are horizontally aligned. Here's a picture of the system

![robot](https://github.com/miguelriemoliveira/RustBot/blob/master/docs/robot.jpg)

## <a name="installation"></a>Installation

First, the Vimba sdk must be downloaded and compiled. We will refer to the folder where the sdk is installed as **$(Vimba_2_0)**.

You may download this sdk from [https://www.alliedvision.com/en/products/software.html](https://www.alliedvision.com/en/products/software.html). Then follow the installation instructions in **$(Vimba_2_0)/Documentation/ReleaseNotes.txt** to install the sdk.

After that, the ros vimba wrapper at [http://wiki.ros.org/avt_vimba_camera](http://wiki.ros.org/avt_vimba_camera) should work. So install the ros wrapper

```bash
git clone https://github.com/srv/avt_vimba_camera.git
catkin_make
```

Install ZMQ Python

```bash
sudo apt-get install python-zmq
```

and google protocol buffers 

```bash
sudo apt-get install python-protobuf
```
Also you should run 

```bash
sudo pip install --upgrade protobuf
```

To use windows based control, you must install the scripts that windows will use through a non-interactive ssh tunnel. To do that, run:

```bash
roscd rustbot_bringug && cd ../scripts
bash ./install_scripts.bash
```
If asked to replace say yes.

## <a name="usage"></a>Usage

To start the complete stereo system, just run 

```bash
roslaunch rustbot_bringup all.launch
```


To launch a single camera use

```bash
roslaunch rustbot_bringup left_camera.launch
```

Note that to run stereo on high resolution images the processing takes a long time. Typically, for 1600x1200 images, our SGBM stereo outputs disparity images at 0.2 Hz. If you want a point cloud to be generated for each disparity image, then, because the point_cloud2 nodelet uses a message filter to get approximately synced image, camera_info and disparity messages, you must run the stereo_processing with a low frame rate and a large queue size. That way the point_cloud2 nodelet can get synced messages and produce the point clouds, e.g.:

```bash
roslaunch rustbot_bringup all.launch fps:=1 queue_size:=50
```

## <a name="connectingssh"></a>Connecting via ssh

To connect to the remote NUC computer

1. Connect an Ethernet cable from your laptop to the switch inside the sensor's box
2. Set a static IP address (the NUC computer has ip 169.254.4.50, so use 169.254.4.51 for example)
3. Ping the NUC Computer to see if everything is ok, i.e. ```ping 169.254.4.50```
4. Launch the ssh connection ```ssh sev@169.254.4.50``` (the password is written in a sticker on the NUC computer)

Note that whenever your ssh tunnel will execute some visual interface application you should start the ssh connection with the _-X_ option, as seen bellow:

4. Launch the ssh connection ```ssh sev@169.254.4.50 -X``` (the password is written in a sticker on the NUC computer)

## <a name="calibration"></a>Calibration

Start the system without the stero processing

```bash
roslaunch rustbot_bringup all.launch do_stereo:=false
```

then startup de calibration

```bash
rosrun camera_calibration cameracalibrator.py --size 7x5 --square 0.03 right:=/stereo/right/image_raw left:=/stereo/left/image_raw right_camera:=/stereo/right left_camera:=/stereo/left --approximate=0.05
```

## <a name="tunningparameters"></a>Tunning Stereo Parameters

To tune the parameters of the stereo algorithm, run

```bash
roslaunch rustbot_bringup all.launch fps:=3 config_stereo:=true
```

and then

```bash
roslaunch rustbot_bringup visualize.launch 
```

Then you can change the parameters an see the effect they have in the disparity map / point cloud in real time.
If you reached a set of parameters you would like to save, do the following:

```bash
roscd rustbot_calibration/calibration/ && rosparam dump stereo_image_proc.yaml /stereo/stereo_image_proc
```
## <a name="savingabagfile"></a>Saving a Bag File

The following commands should be executed from the NUC computer. Thus, if you want to run from a laptop, use an ssh connection as explained in section [Connecting via ssh](#connectingssh).

To record raw data we must first launch the camera drivers (no need to run stereo processing, since this will be done offline)

```bash
roslaunch rustbot_bringup all.launch fps:=10 do_stereo:=false
```

Then, to record messages, run

```bash
roslaunch rustbot_bringup record_raw.launch
```

After stopping (using Ctrl-X) the recorder node, the bag file can be found on the desktop, with the date and hour, e.g., 
_sev_2016-11-24-14-48-30.bag_.

## <a name="playingbackdata"></a>Playing Back Data

```bash
roslaunch rustbot_bringup playback.launch bag:=/home/sev/Desktop/sev_2016-11-03-12-27-31.bag
```

Note: if your bagfile does not contain odometry you can download a dummy odometry from [here](http://criis-projects.inesctec.pt/attachments/download/4003/GPS_IMU_2016-06-14-17-01-59.bag). Then just publish it in addition to the other bags:

```bash
rosbag play GPS_IMU_2016-06-14-17-01-59.bag -l
```
Note2: if your bagfile does not contain odometry info, you can publish dummy /odom messages by running:

```bash
rostopic pub /odom nav_msgs/Odometry "header:
  seq: 0
  stamp:
    secs: 0
    nsecs: 0
  frame_id: ''
child_frame_id: ''
pose:
  pose:
    position: {x: 0.0, y: 0.0, z: 5.0}
    orientation: {x: 0.0, y: 0.0, z: 0.0, w: 0.0}
  covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
    0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
    0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
twist:
  twist:
    linear: {x: 0.0, y: 1.0, z: 0.0}
    angular: {x: 0.0, y: 0.0, z: 0.0}
  covariance: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
    0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
    0.0, 0.0, 0.0, 0.0, 0.0, 0.0]" -r 10
```

Important: If running the rqt_bag, you must press the right mouse button over all topics in the bag file and select play. Otherwise, the topics are not published.



## <a name="stereobagfile"></a>Stereo from a Bagfile

First, you must be playing back recorded data. See section [Playing Back Data](#playingbackdata)

To run the stereo

```bash
roslaunch rustbot_bringup all.launch do_stereo:=true online_stereo:=false 
```

Now you should receive both disparity images /stereo/disparity as well as point clouds /stereo/points2

## <a name="publishingdatazmq"></a>Publishing Data Using ZMQ

Note that in order for this node to work, there must be some node publishing images and point clouds.
This can be done online (not yet funcional) or offline.

To _publish images offline_ follow sections [Playing Back Data](playingbackdata) to publish the recorded images, and also section [Stereo from a Bagfile](#stereobagfile) to publish the disparity maps and the point clouds.


To launch the ZMQ publisher, do:

```bash
rosrun rustbot_translation sev_publisher.py
```

Now launch the ZMQ test subscriber (in Ubuntu python) to see if any images are received:

```bash
rosrun rustbot_translation sev_listener.py
```

If the listener receives images, so should the C# application called [RustBoxCSharp](https://github.com/carlosmccosta/RustBotCSharp) as long as its propperly configured.

## <a name="findingcameraip"></a>Finding IP Address of Cameras

If the network has no dhcp service, the cameras fall back to a default ip address, which is:

169.254.133.197 Left Camera
169.254.4.55 Right Camera
169.254.4.50 NUC computer, select "SEV local network" in the network manager

To find out the ip address and the id of your cameras. From the **$(VimbaPath)/examples/ListCamera** folder, run

```bash
cd $(Vimba_2_0)
/Tools/Viewer/Bin/x86_64bit/VimbaViewer
```

You shoud see the VimbaViewer gui with all connected cameras with their id (guid). 
If no cameras appear then you may be in a foreign subnet, i.e. have an ipaddress in a different segment. In this case run 

```bash
cd $(Vimba_2_0)
sudo -E /Tools/Viewer/Bin/x86_64bit/VimbaViewer
```
and you should see the cameras

![vimba_viewer](https://github.com/miguelriemoliveira/RustBot/blob/master/docs/vimba_viewer.png)

Select a camera and then, on the right side pane, select All propperties and search for "IP Address" to find out the the ip address.

![get_ipaddress](https://github.com/miguelriemoliveira/RustBot/blob/master/docs/get_ipaddress.png)

You may edit the file **launch/mono_camera.launch** and 
 

By setting the correct guid and ip of your camera.

You can get the guid and ip of your camera by using the ListCamera or VimbaViewer binaries in

Vimba_2_0/VimbaCPP/Examples/Bin/x86_64bit

## <a name="zeromqtutorial"></a> ZeroMQ + Google Protocol Buffers Tutorial

To launch the publisher

```bash
rosrun rustbot_translation example_publisher.py
```

and to receive the subscriber

```bash
rosrun rustbot_translation example_listener.py
```

## <a name="compilemessages"></a>Compile Google Protocol Buffers Messages

To compile all the required messages use the script:

```bash
roscd rustbot_translation && ./compile_proto.sh
```

If you want the compile individual mesages for python use something like:

```bash
roscd rustbot_translation
```

```bash
protoc -I=./msgs -I=/home/mike/workingcopy/protoc-3.1.0-linux-x86_64/include/google/protobuf/ --python_out=src/ msgs/SEVData.proto
```

This assumes you have the protoc version 3 binaries in /home/mike/workingcopy/protoc-3.1.0-linux-x86_64/

You may download [here](https://github.com/google/protobuf/releases/tag/v3.1.0). 

Add a link to this version of protoc.

```bash
mkdir ~/bin && ln -s ~/workingcopy/protoc-3.1.0-linux-x86_64/bin/protoc protoc
```

Make sure the ~/bin folder is the first in your path, otherwise you may be using another version of protoc.
You may confirm this with

```bash
protoc --version
```

See [this](https://developers.google.com/protocol-buffers/docs/pythontutorial) for additional info.
