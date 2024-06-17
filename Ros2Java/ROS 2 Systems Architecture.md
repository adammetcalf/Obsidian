As a basic overview, ROS 2 consists of nodes which may communicate. Nodes may be considered as endpoints, topics may be considered as a bus and [messages](obsidian://open?vault=AdamNotes&file=ROS%202%20Messages%20and%20DDS)are data which are transported on the bus:

![[ROS2Basic.png]]

ROS2 is built upon a transport protocol called [DDS (Data Distrubution Service)](https://design.ros2.org/articles/ros_on_dds.html):

![[ROS2Architecture.png]]

If we look at the basic OSI model and ROS 2 implementation:
![[BasicOSI.png]]

In more detail:
![[DetailedOSI.png]]
