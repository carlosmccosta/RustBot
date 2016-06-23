# ProjectSEV
Software tools for the Sistemas Embarcados de Vistoria (SEV) project

## Table of Contents

* [The Robot](#therobot)
* [Installation](#installation)
* [Usage](#usage)
* [Calibration](#calibration)
* [Calibration Setup](#calibrationsetup)

## <a name="therobot"></a>The Robot


## <a name="installation"></a>Installation

First, the Vimba sdk must be installed and compiled. We will refer to the folder where the sdk is installed as **$(Vimba_2_0)**

You may download this sdk from [https://www.alliedvision.com/en/products/software.html](https://www.alliedvision.com/en/products/software.html). Then follow the installation instructions in **$(Vimba_2_0)/Documentation/ReleaseNotes.txt** to install the sdk.

After that, the ros vimba wrapper at [http://wiki.ros.org/avt_vimba_camera](http://wiki.ros.org/avt_vimba_camera) should work. Follow these steps.

Install the ros wrapper

```bash
git clone https://github.com/srv/avt_vimba_camera.git
catkin_make
```

Find out the ip address and the id of your cameras. From the **$(VimbaPath)/examples/ListCamera** folder, run

```bash
cd $(Vimba_2_0)
/Tools/Viewer/Bin/x86_64bit/VimbaViewer
```

You shoud see a gui that shows the id and ip address of all connected cameras.

![vimba_viewer](https://github.com/miguelriemoliveira/RustBot/blob/master/docs/vimba_viewer.png)



You may edit the file **launch/mono_camera.launch** and 
 

By setting the correct guid and ip of your camera.

You can get the guid and ip of your camera by using the ListCamera or VimbaViewer binaries in

Vimba_2_0/VimbaCPP/Examples/Bin/x86_64bit
