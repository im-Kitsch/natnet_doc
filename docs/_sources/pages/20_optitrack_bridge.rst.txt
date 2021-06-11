Use optitrack bridge
====================


In Motive open ``view->Data Streaming Pane``, click on the 3 dots in the right upper corner, 
then ``Show Advanced``

1. rename or delete the file **CATKIN_IGORE** in optitrack/optitrack_bridge
   
2. Adjust the launch file in `optitrack_bridge/optitrack.launch`
   
   - ``host_ip`` -> MultiCastIpAddress. Default is ``239.255.42.99``
   
   - ``host_port`` -> DataPort. Default is ``1511``
3. Disable ``Assets Markers`` in Motive DataStreaming Pane. 
4. Start the ros bridge::
   
    $ roslaunch optitrack_bridge optitrack.launch

5. Check the rigid body TFs in rviz.

.. note::
   - You should set the coordinate setting in Motive as ``Y-UP``.
   - Some warning like lost frame, will be because the rigid body doesn't move (the position stays the 
     same as the last frame) or invalid or any reason.....