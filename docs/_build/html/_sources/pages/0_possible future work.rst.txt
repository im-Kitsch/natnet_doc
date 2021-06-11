Advices future work
===========================

Software latency
---------------------------
It seems that motive has not enough information about time format. 
Otherwise it is possibly to use it and estimate the software latency. 
The official API has a estimation of transpose latency, including camera latency and transpose latency...
But parsing the data packet has not enough description about this.
 

Data Description
----------------------------
The message for data description is also very useful. But currently it is not needed. It is worth to parse also data description.
It seems it should need firstly send corresponding message to Motive or the DataDescription methods are sent after initial 
communicate but not sent regularly like dataframe.


Linux Library
-----------------------------

Besides manually parse the data from Multicast via low level Linux APIs, NatNet provide pre-compiled library since Motive 3.0. 
Using there interface it's also possible to parse the data and communicate Motive in Windows-side. 

To use the API,
In `NatNet SDK <https://optitrack.com/support/downloads/developer-tools.html/>`_, 
download the Linux version. Follow the instructions, build and run it. But there is 
not enough/well documented documentations for that.

It is also contained in *Reference/natnet_sdk* folder. 
It is good to use it for developer to validate communication and compare with ROS pkg.





