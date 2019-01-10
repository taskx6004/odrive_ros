# odrive_ros for spin_hokuyo

dynamixel内部包含串口驱动和电机控制，串口发布motor_states作为rostopic监控；电机控制订阅外部的Twist命令，发给电机进行控制，同时输出JointState，TF给pcl registration和point cloud采集程序

hokuyo_ws下其他程序接受JointState，TF，进行点云采集和粗对准，并输出为MAP

所以启动方法一目了然，给rostopic中的Twist赋值即可，如给一个10Hz更新率，2.3m/s 的速度：rostopic pub /cmd_vel geometry_msgs/Twist -r 10 -- '[2.3, 0.0, 0.0]' '[0.0, 0.0, 0.1]'

本程序odrive_ros完成了对JointState和TF的打包，在使用的时候需要把dynamixel中的dynamixel_msg文件夹保留，里面有这些topic的信息。

## ISSUE
这个信息是pip install odrive版本内部与现有python版本不一致导致的，但不会出错
```
Exception in thread Thread-7:
Traceback (most recent call last):
  File "/usr/lib/python2.7/threading.py", line 801, in __bootstrap_inner
    self.run()
  File "/opt/ros/kinetic/lib/python2.7/dist-packages/rospy/timer.py", line 235, in run
    self._callback(TimerEvent(last_expected, last_real, current_expected, current_real, last_duration))
  File "/home/parallels/catkin_ws/src/odrive_ros/src/odrive_ros/odrive_node.py", line 362, in timer_odometry
    vel_l = self.driver.left_axis.encoder.vel_estimate  # units: encoder counts/s
  File "/usr/local/lib/python2.7/dist-packages/fibre/remote_object.py", line 239, in __getattribute__
    return attr.get_value()
  File "/usr/local/lib/python2.7/dist-packages/fibre/remote_object.py", line 72, in get_value
    buffer = self._parent.__channel__.remote_endpoint_operation(self._id, None, True, self._codec.get_length())
  File "/usr/local/lib/python2.7/dist-packages/fibre/protocol.py", line 309, in remote_endpoint_operation
    raise ChannelBrokenException()
ChannelBrokenException
```
## Usage

As of September 7, 2018 the main ODrive repository doesn't support Python 2.7, so you will need to ensure the version you install has [my compatibility patch](https://github.com/madcowswe/ODrive/pull/199). Said patch is also is a bit rough around the edges, you will need to use the odrivetool for configuration according to the ODrive setup docs with `python2 odrivetool` instead of running it directly.

To install:
```sh
git clone https://github.com/neomanic/ODrive -b py27compat
cd ODrive/tools
sudo pip install monotonic

# sudo python setup.py install # doesn't work due to weird setup process, so do the following:
python setup.py sdist
sudo pip install dist/odrive-0.4.2.dev0.tar.gz
```

then `rosrun odrive_ros odrive_node` will get you on your way. 

There is a sample launch file showing the use of the wheelbase, tyre circumference, and encoder count parameters too, try `roslaunch odrive_ros odrive.launch` or copy it into your own package and edit the values to correspond to your particular setup.

If you want to test out the driver separate from ROS, you can also install as a regular Python package.

```sh
roscd odrive_ros
sudo pip install -e .
```

Fire up a Python shell. See below for the import which exposes the ODrive with setup and drive functions. You can use either ODriveInterfaceAPI (uses USB) or ODriveInterfaceSerial (requires the /dev/ttyUSBx corresponding to the ODrive as parameter, but not tested recently).

```python
from odrive_ros import odrive_interface
od = odrive_interface.ODriveInterfaceAPI()
od.connect()
od.setup()          # does a calibration
od.engage()         # get ready to drive
od.drive(1000,1000) # drive axis0 and axis1
od.release()        # done
```

## Acknowledgements

Thanks to the ODrive team for a great little bit of hardware and the active community support. Hope this helps!

- [ODrive homepage](https://odriverobotics.com)
- [ODrive getting started](https://docs.odriverobotics.com)
- [ODrive main repo](https://github.com/madcowswe/ODrive)

Thanks to Simon Birrell for his tutorial on [how to package a Python-based ROS package](http://www.artificialhumancompanions.com/structure-python-based-ros-package/).

